---
title: "CIS Controls at Home - Overview"
date: 2024-12-08T02:00:00
tags:
  - cis
  - concepts
categories: 
  - cybersecurity
 
draft: true
url: "cis-at-home-overview"
summary: "Overview of Plan to implement the CIS Controls v8.1 at home network"
description: "Overview of Plan to implement the CIS Controls v8.1 at home network"

---


In an effort improve in my understanding cybersecurity posture/defense, I will implement the CIS Controls (v8.1) on my small home enviroment.  I will try to go through all 18 controls - focusing on IG1 first. Perhaps later I will return to go through or examine IG2 and IG3 more closely.

My plan is to do 1 post per Control section, summarizing what I did to accomplish this task, or describing what prevented me from accomplishing it.  I also will to provide some context of the purpose of each control and the action taken.

In an enterprise settings, there is a lot of buy-in needed by leadership and all parties to implement these controls.  By doing it in my small home network, I can become more familar with the controls and the underlining principles.

By documenting on this blog, I hope to sound down some of these priniciples as well as improve in technical know now.

I intend to move through each control as quickly as possible, yet I am open to use this to drive some other areas of focus in my learning as much as possible.  At the outset I will look for oppurtinties to use these controls to improve in these areas
- python
- linux
- automation
- API
- cloud services

## More About CIS Controls


### Who Created?
Center for Internet Security - a worldwide community.

### What Are They?
security best practises to protect small office to enterprise networks.
designed to be based on best practices, attacker techniques and community input.
Not vendor specific, but

they also provide baselines and hardening guideline for various devices, Operating Systems and vendors.

Divided up into 18 large Controls - or major category.
Each category is divided into various safeguards - or a specfic task/prodedure/policy to be performed.  Each safeguard is associated with a Security Function such as govern, protect, identity, respond, or recover.

### What are the IG (Implmentation Groups)?

To help with various levels of enterprises and to prioritieze the most critical safeguards, 3 different Implementation Groups (IG) are identified.

IG1/2/3 each identifity 3 levels of safeguard - each with increasing effectiveness and complexity
IG1 - home office, limited staff, essiential cybersecurity hygene
IG2 - it/security staff exists.  Enterprise with data and reputational concerns to be addresss
IG3 - large-scale operation with security teams handling all aspects of infomration secrurity.

While going through all 18 CIS Controls, and organization can start with completing all the IG1 safeguards, then returning to IG2 and IG3.  
This increase effectifiveness and security by - prioritization.

Some oranizations may not currently have the capablity of completed all the IG2/3 safeguards.



### Why Use CIS Controls?
align with different frameworks, so by implenting CIS controls, will already be accomplishing many of the larger scale enteprise frameworks (NIST/ISO27001)


My impression of CIS controls, as opposed to other cybersecurity frameworks, is that they are more action oriented, with focus on most important steps to defend.



## Action Plan
- Downloaded the most recent CIS Controls v 8.1 in excel format
- Edited the Excel spreadsheet to filter out IG2/3 - so one focused on IG1
- Added additional columns to spreadsheet for each CIS Safeguard:
Date started, Date Completed, Lessons Learned, References


## Progress so far...


CIS Control 1 Inventory and Control of Enterprise Assets 