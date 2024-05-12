---
title: "Temple Social Engineering Competition 2024"
date: 2024-04-21
tags: ["Writeup", "Competitions", "LaTeX", "Social Engineering"]
---

I recently competed in Temple University's [Social Engineering Competition](https://sites.temple.edu/socialengineering/) for the first time, along with some other [CCSO](https://psuccso.org) members. We were able to take first place, with our B team placing 3rd! This competition is run by the [CARE Lab](https://sites.temple.edu/care/) at Temple, and this year's theme was employment and tax scams. The competition format was pretty different than anything I'd done in the past and had us meeting with a client and helping her through a potential employment scam.

![2024 Winners](content/posts/competitions/temple-sectf-24/featured.png)

Overall this was a great opportunity to learn more about social engineering and common scams, and I'm really happy with the final product my team and I were able to deliver. I also decided to take this as an opportunity to learn LaTeX as I move into graduate school, so I've provided a repo with our full code and deliverables as well.

## Overview

The competition took place over three days and had us interact with a fictional client named Sam. Sam's received an offer to interview for a remote job, but her friends are concerned it's a scam. As part of the fraud-fighting team, we were tasked with helping Sam through the hiring process and delivering comprehensive reports regarding the legitimacy of the job offer. We ended the competition with a final brief to Sam outlining the evidence we'd collected, and our recommendations for how she should handle the situation.

## Timeline

{{< timeline >}}

{{< timelineItem header="Initial Client Meeting" icon="location-dot" badge="Day 1">}}
The initial meeting had us speaking with Sam's friends, who were initially concerned about the potential scam. Our team took the opportunity to gather more information about both Sam and the job opportunity. We learned that Sam was an unemployed student who posted her resume on a job searching site, soon after she was contacted by someone claiming to be from the company HealthComp with an offer for a text-based virtual interview. The email had a lot of red flags, it offered multiple different jobs, all of which were remote. The contact provided didn't match the email sender, it was riddled with typos, and none of the jobs existed on the company's website.
{{< /timelineItem >}}

{{< timelineItem header="Sam's Chat-based Interview" icon="comment" badge="Day 2">}}
Kicking off day two, the team was tasked with helping Sam through her text-based interview with the prospective employer. Once again, there were many red flags. Initially, Sam was told the interview would be with David Bondeson, while the actual interview was conducted by Gavin Manley. Gavin interchangeably referred to the company as both Omnicell (not the original company name) and HealthComp, and was generally inconsistent with details. The interviewer also responded to the messages insanely fast and behaved like a chatbot. Sam gets told that should she receive the position, she'll need to purchase a laptop and home office materials the company will reimburse her for. Our team identified this as the likely vector to steal money from Sam, along with any other additional information the scammers would attempt to obtain throughout the hiring process.
{{< /timelineItem >}}

{{< timelineItem header="Sam Gets the Job" icon="envelope" badge="Day 2">}}
Shortly after the interview, Sam has a follow-up chat with Gavin where she's offered the job. She's asked to fill out a W4, a job application, and provide a passport photo for an ID card, none of which makes a lot of sense. She's already obtained the position, and an ID card wouldn't be needed for a fully remote position. This entire process was extremely consistent with a scam report the team found through OSINT, that had the same premise and point of contact.
{{< /timelineItem >}}

{{< timelineItem header="Formal Debrief" icon="star" badge="Day 3">}}
At this point we were given the opportunity to share our findings and recommendations with Sam, hopefully convincing her to take the appropriate course of action. We outlined our OSINT findings about the company, scam etc. and all the relevant red flags from her interactions with the employer/scammer. Our team concluded that it was in fact a scam (who could've ever seen this coming), and directed same to cease communication and report the scam. Most importantly, we wanted to ensure that Sam didn't share any sensitive PII or payment information with the scammer.
{{< /timelineItem >}}

{{< /timeline >}}

## Deliverables

Our team was tasked with providing several deliverables throughout the competition. I've outlined them below and provided a GitHub repo with our full code and compiled reports/presentations. Both days had deliverables due at midnight, and we gave our presentation prepared on day 2 the morning of day 3.

**Day 1 Deliverables:**

* Employer/scammer profile (utilize OSINT)
* Timeline of events
* Persuasive methods used across the timeline
* Red flags
* Plan to advise Sam on Day 2

**Day 2 Deliverables:**

* OSINT Findings
* Red flags
* Timeline of events
* MITRE ATT&CK Framework mappings across the timeline
* Apply the NIST Phish Scale worksheet to all emails received
* List of red flags to identify employment scams
* Checklist of how Sam should move forward
* General checklist for victims of tax/employment fraud scams
* Gameplan for advising Sam on day 3
* Presentation for Day 3

{{< github repo="lfgberg/Temple-SECTF-24" >}}
