---
title: "CIS Controls at Home: Control 1 IG1 - Inventory and Control of Enterprise Assets"
date: 2025-03-11T02:00:00
tags:
  - cis
categories: 
  - cybersecurity
 
draft: false
url: "cis-at-home-control1-ig1"
summary: "Summary of steps and results implementing CIS Control 1 IG1"
description: "Summary of steps and results implementing CIS Control 1 IG1"

---

[CIS v8.1 Controls at Home - Overview]({{< relref "CIS-8.1_Controls-At-Home-Overview.md" >}})

# CIS Control 1 = Inventory and Control of Enterprise Assets
CIS Control 1 objective is to actively manage all enterprise assets connected physically, virtually, remotely or in cloud environments. 
This allows provides knowledge of all assets that exist, and need to be monitored and protected.
You can not secure what you don't monitor.
This also will enable the identifying and removal of unauthorized or un-managed devices from the network.

## CIS Safeguard 1.1, IG1 = Establish and Maintain Detailed Enterprise Asset Inventory
This safeguard is under the "Identity" Security Function. The task is to create a create and maintain a detailed list of all assets (any device able to store and compute data).
Accurate information for each asset to identify it, its owner and wether it is approved. 

IG1 states to review and update the inventory bi-annually or more frequently.

### Safeguard Implementation


To accomplish this safeguard, I utilized the Fortigate 70F I have connected to my network.  I enabled "device detection" on the interface that connects to my eero home wifi and the interface that connect to my lab.  This allowed the fortinet to discover most of the devices connected to my home network via ARP, or other means.

I then utilized fortinet API call "api/v2/monitor/user/device/" to get all collect all these devices and capture the output.  
I used AI to help write a python script that would use the Fortinet API to collect the info (returned as JSON) and then convert that into CSV file and save to my computer.  To accomplish this, AI recommended using the requests and json libraries, as well as the pandas library (which is for working with csv).
I generated an API token on the fortigate for this, restricted only to private lab IPs.

The script had to run in an virtual environment to prevent some issues installing libraries with homebrew.  I called the python script "get-devices.py".  I hard coded the Fortigate IP, management port, and API token into the script.

It generated a csv file based on the returned API JSON.  It had more fields than needed, and some of the data/descriptions was inaccurate or incomplete.

I then cleaned up the csv to create an accurate and readable device inventory list.  I modified the columns, and only added IPs for static ip devices.   I also gave a more identifiable name for each device.  

The CIS safeguard says to also include items that are in cloud environments, so I added my clab-vm that lives in GCP, as it was not picked up by the fortigate.

I added the columns "Owner" and "Approved" and updated for each device based on safeguard instructions.  I dated the excel document, and send reminder for 6 months to review this again.

## CIS Safeguard 1.2, IG1 = Address Unauthorized Assets

Ensure a process exists to address unauthorized assets and remove them from the network, deny the device from connecting, or quarantining the asset.

### Safeguard Implementation

Created a document called "PLAB Security Policy" which will be used to maintain high-level security policy for PLAB.  A policy entry was created for "Assets".

Created a document called "PLAB Security Processes and Procedures".  Created a process for the review of inventory every Monday by viewing devices on the Fortigate.

This process is manual, and lacking history and visibility, but it is start of the journey.

