---
title: "Deloitte Cyber Threat CTF 2023"
date: 2023-11-01
categories: ["Security"]
tags: ["CTF", "Writeup"]
---

This was my first time participating in Deloitte's Cyber Threat Competition, which consists of two parts. The first round consisted of a brief multiple choice quiz on security and business topics, and a CTF, which I'll be walking through in this post. The top participants from each participating school are invited to attend round 2 at Deloitte University. I was able to place 9th out of 160 participants, moving onto round 2, and completing all but 2 challenges within the given timeframe.

## Introduction Challenge

Challenge: `There is a flag hidden on the web page of this challenge`

Browsing to the site provides us with a relatively blank page:

![Website](security/deloitte23/intro-site.png)

If we view the page source, there's a series of br elements padding the page, with taunting comments. The flag is in a comment at the end of the page:

![Flag](security/deloitte23/inspect-element-flag.png)

## Weak Crypto

Challenge:

```text
We were recently hacked, so we needed to take our entire login application offline. All users who access the site are forced to be anonymous users only based on their session token.  
  
Can you verify that our fix is effective?  
  
**Note:** The login functionality and form are actually disabled, there is no brute forcing required to solve this challenge; focus on the session token
```

Browsing to the page provides us with a disabled login form and a link to the route /api/user which we can use to verify the currently logged-in user:

![Admin Panel](security/deloitte23/admin-panel.png)
![/api/user](security/deloitte23/api-user.png)

Looking at the cookies reveals that the site stores a session token, which is a base64 encoded JWT token:

![Session Token](security/deloitte23/session-token.png)

Based on the challenge title, we can assume they don't perform any validation etc. on the session token. Decoding the the token presents us with the value `{"user": "anonymous"}`, to get the flag, change the value to `{"user": "administrator"}`, and replace the current session with your new token. Browsing to api/user shows that we're authenticated as the administrator, and presents us with the flag:

![Flag](security/deloitte23/token-flag.png)

## Default Administrator Password

Challenge: `It seems that someone forgot to change their vendor default password, can you find it?`

We're presented with an Apache Geronimo login page:

![Geronimo Login](security/deloitte23/geronimo-login.png)

A quick search reveals that the default creds are `system/manager`, and logging in presents us with the flag:

![Flag](security/deloitte23/geronimo-flag.png)

## Call an Ambulance

Challenge: `Our server is vulnerable to a well known attack. What was it called? Shellshock? Poodle?`

An nmap scan of the host reveals the following open port (utilize `-p-` to find the uncommon port):

```text
47238/tcp open  ssl/http nginx 1.5.12  
| tls-nextprotoneg:    
|_  http/1.1  
|_http-server-header: nginx/1.5.12  
|_ssl-date: TLS randomness does not represent time  
| ssl-cert: Subject: commonName=10.1.2.8/organizationName=DeloitteCTF/countryName=US  
| Not valid before: 2015-02-20T13:57:08  
|_Not valid after:  2015-02-27T13:57:08  
|_http-title: Did not follow redirect to https://10.6.0.2:47238/
```

The challenge name, as well as service versions, seem to indicate that this server is vulnerable to heartbleed, a vulnerability in OpenSSL that allows an attacker to gain access to portions of memory that should be restricted. Using the `scanner/ssl/openssl_heartbleed` module in metasploit, with `verbose` set to `true`, we can see the flag exposed in the server's response to our request:

![msf Options](security/deloitte23/msf-options.png)
![Heartbleed Output](security/deloitte23/heartbleed-output.png)

## Evil Eval

Challenge: `Allowing users to eval() JavaScript should be fine right?`

Browsing to the site we're presented with an input field to have our code evaluated, entering basic expressions like `1===1` results in a success, but more complex requests that use other functions like `alert("test")` get blocked. Successful eval requests don't result in any visual or console output.

![Eval Page](security/deloitte23/eval.png)

After doing some more investigation, using `html += CODE` will append the result of our evaluated code to the page, and we can utilize `require("child_process").execSync('COMMAND').toString()` will run bash commands on the system. We can use `html += require("child_process").execSync('find / --name flag*').toString();` to search the file system for a file with flag in the name, and `html += require("child_process").execSync('cat /src/flag.txt').toString();` to find the flag:

![ls Output](security/deloitte23/ls-eval.png)
![Flag](security/deloitte23/eval-flag.png)

## Simple Nslookup Tool V2

Challenge:

```text
Version 1 of our nslookup tool was vulnerable to command injection. As a fix I blocked the common command execution tricks (;&$><`\!).  
  
Can you test version 2 for me and make sure it is not possible to call any other binaries?
```

Browsing to the site we're presented with an input field to have our query evaluated, and the output is presented in a field below:

![nslookup](security/deloitte23/nslookup.png)

Although they block common execution tricks like `(;&$><\!)`, they don't block `|`. We can input any query followed by a pipe and arbitrarily execute commands. A quick ls reveals `flag.txt` in the directory:

![ls Output](security/deloitte23/nslookup-ls.png)
![Flag](security/deloitte23/nslookup-flag.png)
