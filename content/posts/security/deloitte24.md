---
title: "Deloitte Cyber Threat Competition 2024"
date: 2024-02-17
categories: ["Security"]
tags: ["CTF", "Writeup"]
---

![Victory Photo; RAHHHHHHHH](security/dctc24/victory.JPG)

I recently participated in Deloitte's Cyber Threat Competition for the first time, placing second. The competition consisted of two rounds: round 1 had a security questionnaire and CTF, and round 2 had another brief CTF as well as an incident response wargame and presentation portion. The top 2 students from each school and other high-scoring participants moved on to round two where teams were randomly assigned across universities.

Overall this was a great opportunity to go out of my comfort zone and focus more on the business aspects of cybersecurity, I really enjoyed working with my teammates and getting to travel to Deloitte University. I've detailed the CTF challenges from round 1, as well as the IR scenario from round 2.

![Team 8](security/dctc24/team-8.JPG)

## Round 1 - CTF

### Introduction Challenge

Challenge: `There is a flag hidden on the web page of this challenge`

Browsing to the site provides us with a relatively blank page:

![Website](security/dctc24/intro-site.png)

If we view the page source, there's a series of br elements padding the page, with taunting comments. The flag is in a comment at the end of the page:

![Flag](security/dctc24/inspect-element-flag.png)

### Weak Crypto

Challenge:

```text
We were recently hacked, so we needed to take our entire login application offline. All users who access the site are forced to be anonymous users only based on their session token.  
  
Can you verify that our fix is effective?  
  
**Note:** The login functionality and form are actually disabled, there is no brute forcing required to solve this challenge; focus on the session token
```

Browsing to the page provides us with a disabled login form and a link to the route /api/user which we can use to verify the currently logged-in user:

![Admin Panel](security/dctc24/admin-panel.png)
![/api/user](security/dctc24/api-user.png)

Looking at the cookies reveals that the site stores a session token, which is a base64 encoded JWT token:

![Session Token](security/dctc24/session-token.png)

Based on the challenge title, we can assume they don't perform any validation etc. on the session token. Decoding the the token presents us with the value `{"user": "anonymous"}`, to get the flag, change the value to `{"user": "administrator"}`, and replace the current session with your new token. Browsing to api/user shows that we're authenticated as the administrator, and presents us with the flag:

![Flag](security/dctc24/token-flag.png)

### Default Administrator Password

Challenge: `It seems that someone forgot to change their vendor default password, can you find it?`

We're presented with an Apache Geronimo login page:

![Geronimo Login](security/dctc24/geronimo-login.png)

A quick search reveals that the default creds are `system/manager`, and logging in presents us with the flag:

![Flag](security/dctc24/geronimo-flag.png)

### Call an Ambulance

Challenge: `Our server is vulnerable to a well known attack. What was it called? Shellshock? Poodle?`

An Nmap scan of the host reveals the following open port (utilize `-p-` to find the uncommon port):

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

![msf Options](security/dctc24/msf-options.png)
![Heartbleed Output](security/dctc24/heartbleed-output.png)

### Evil Eval

Challenge: `Allowing users to eval() JavaScript should be fine right?`

Browsing to the site we're presented with an input field to have our code evaluated, entering basic expressions like `1===1` results in a success, but more complex requests that use other functions like `alert("test")` get blocked. Successful eval requests don't result in any visual or console output.

![Eval Page](security/dctc24/eval.png)

After doing some more investigation, using `html += CODE` will append the result of our evaluated code to the page, and we can utilize `require("child_process").execSync('COMMAND').toString()` will run bash commands on the system. We can use `html += require("child_process").execSync('find / --name flag*').toString();` to search the file system for a file with flag in the name, and `html += require("child_process").execSync('cat /src/flag.txt').toString();` to find the flag:

![ls Output](security/dctc24/ls-eval.png)
![Flag](security/dctc24/eval-flag.png)

### Simple Nslookup Tool V2

Challenge:

```text
Version 1 of our nslookup tool was vulnerable to command injection. As a fix I blocked the common command execution tricks (;&$><`\!).  
  
Can you test version 2 for me and make sure it is not possible to call any other binaries?
```

Browsing to the site we're presented with an input field to have our query evaluated, and the output is presented in a field below:

![nslookup](security/dctc24/nslookup.png)

Although they block common execution tricks like `(;&$><\!)`, they don't block `|`. We can input any query followed by a pipe and arbitrarily execute commands. A quick ls reveals `flag.txt` in the directory:

![ls Output](security/dctc24/nslookup-ls.png)
![Flag](security/dctc24/nslookup-flag.png)

## Round 2

![Team Introductions](security/dctc24/team-brief.JPG)

### Wargame

#### Scenario

During the incident response wargame, each team works through a series of scenarios following a single company and cyber incident. This year's scenario had teams serving as consultants for "Green Dot Destinations" (GDD), a fictional hotel company with 50 hotels throughout the US China and Japan. The company's most critical asset was its Property Management System (PMS) which centrally controlled room bookings, keycard access, payment processing, and other critical functions for the business. GDD notably outsourced all customer support to a 3rd party which provided a chat and phone lines.

Teams were asked to divide themselves into the following roles:

* Cyber Strategist - Responsible for risk assessment, managing human and financial capital, and reporting to shareholders
* Crisis Communications Specialist - Responsible for coordinating communication with the media, maintaining customer satisfaction, and internal communication
* Data Privacy Specialist - Responsible for managing legal and regulatory concerns regarding data privacy
* Detect and Respond Specialist - Responsible for forensic investigation and technical response to cyberattacks

I served as our team's Detect and Respond Specialist, focusing on covering the more technical portions of the scenario along with our Data Privacy Specialist.

#### Incident

Throughout the wargame, the following occurred over 8 scenarios:

1. Our consulting team was brought onto GDD's project
2. An attacker conducted a DDoS attack that took down GDD's public website and impacted PMS performance
3. Ransomware is deployed across the network, attackers request a 30 million dollar ransom via Bitcoin
4. GDD's daily operations are severely impacted due to the PMS takedown, customers are unable to access rooms, etc.
5. Customer and employee data is posted for sale on a tor site by the attacker
6. The Wall Street Journal writes an article about the incident and data breach, calling for a public update from GDD
7. The attacker compromises GDD's 3rd party support chat, taunting customers seeking help
8. A full forensic report is delivered after collaborating with law enforcement

The forensic report revealed the full story behind the attack. A malicious actor was able to socially engineer a GDD employee via vishing, gaining admin access to the company's infrastructure. They then exfiltrated customer/employee information and began deploying ransomware. The DDoS attack on GDD's website was conducted to distract from the larger campaign.

This realistic scenario was fun and engaging to work through, during the exercise we had very little information about the technical controls or IC/BC policies GDD had in place. As a result during each scenario, we had to work through every possibility ensuring that we thought through not only technically how to respond to the incident but also how to mitigate the reputational fallout and manage the impact on critical business functions as a result of the PMS downtime.

### Brief

![Yappin Away](security/dctc24/yappin.JPG)

After the wargame, teams had 3 hours to prepare a brief for GDD's C-Suite Executives to update them on the recent attack. We weren't able to bring any materials from the wargame out of the room, so detailed note-taking was critical to have all the information.

Teams were asked to:

1. Summarize the incident
2. Describe any immediate actions taken or decisions made
3. Detail the short and long-term impacts of the incident

The brief had a rapid turnaround time with little time to practice, and a strict 8-minute time limit excluding questions. I provided a timeline of the attack and focused on our immediate actions taken during the wargame. My teammates detailed the technical and policy changes we recommended, and the financial/operational/reputational impacts of the attack.

Overall I enjoyed the Cyber Threat Competition and found it to be a fun counterpart to the more technical competitions like CCDC/CPTC. My teammates and tabe judge were wonderful to work with and I learned more about the nontechnical aspects of the incident response cycle.
