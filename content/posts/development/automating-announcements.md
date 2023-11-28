---
title: "Automating Club Announcements"
description: "Automating CCSO club announcements with Microsoft PowerAutomate"
date: 2023-09-03
categories: ["Development"]
tags: ["PowerAutomate", "Automation"]
---
While serving as the secretary for CCSO, one of my weekly tasks was to aggregate everything the club was doing and distribute that amongst our various platforms. This quickly became very tedious, and to have more time to devote elsewhere, I decided to automate the email announcement process with Microsoft PowerAutomate. PowerAutomate is a fairly simple automation platform that allows users to use drag-and-drop programming to automate repetitive mundane tasks. PowerAutomate has very robust integrations with other Microsoft applications like Sharepoint and Outlook, which makes it a perfect choice for managing the club's email list.

## Goals

When creating this announcement setup I wanted to make sure it accomplished the following goals:

* Integrates with the club's Sharepoint page
* Provides a uniform neat format for emails
* Requires human approval before sending an email

## Setting Up The Workflow

When creating a workflow, PowerAutomate provides a couple of different choices for how workflows are triggered. I chose to create two different Scheduled Cloud Flows, one of which runs every Sunday to remind Exec Board Members to input anything they'd like announced, and one that runs every Monday which creates and sends the final email to our mailing list.

### Reminder Workflow

The Sunday night reminder flow is fairly simple, it's set up to run every Sunday afternoon for the semester and sends a reminder from our club email to all of the Exec Board members to place all of their announcements in the Scrum Board.

![Reminder Flow](automating-announcements/reminder-flow.png "Reminder Workflow")

### Announcements Workflow

For the announcements workflow, the first thing is to aggregate a list of all the things to announce from the club's Sharepoint Page. I created 3 different buckets on a planner to sort announcements into either meeting reminders, other events, or WIP. Anything placed in the WIP Bucket will not be sent in the email, it'll instead serve as a place for drafts or reminders about formatting.

![Announcements Planner](automating-announcements/scrum-board.png "Sharepoint Announcements Planner")

After listing the tasks from the announcements planner, I created a series of conditionals to append the task's title and description to an array of the appropriate category (meeting reminders or events), filtering by bucketID.

![Building our lists](automating-announcements/conditionals.png "Conditionals to filter tasks and build out our arrays")

With the arrays populated and joined, I created an email template that would display them in a bulleted format, this draft email is sent to club officers before a final approval process is started.

![Email Draft](automating-announcements/email-template.png "Email draft template")

Lastly, before the final email is sent to the mailing list, one of the officers must approve the draft email. All the officers are sent an approval request via email, whoever approves/rejects it first either causes the email to get sent to the mailing list or be rejected for editing.

![Approval](automating-announcements/approval.png "Approval and final conditional")