---
title: "Netlab BGP Labs Series: Getting Started with GitHub Codespaces"
date: 2024-11-01T01:00:00
tags:
  - netlab
  - bgp
  - bgplabs
  - codespaces

categories: 

draft: false
url: "netlab-bgplabs-overview"
summary: "Overview of Netlab, BGPLabs and GitHub Codespaces"
description: "Overview of Netlab, BGPLabs and GitHub Codespaces"
---



I got wind of Ivan explaining netlab and his bgplabs.  I thought it was a great way to strengthen my knowledge of BGP and also learn by doing with GitHub, virtualization and some new Network OS vendors.

I will give some overview thoughts here, and then do a different post for each lab I complete

## What are GitHub Codespaces?

GitHub Codespaces is a feature that allows you to create and run an vm instance with many common development tools pre-installed. This means you can execute code from GitHub repository immediately, making it ideal for testing and development. Each Codespace can be customized to suit your needs, providing a tailored development environment.

## Understanding Netlab and BGP Labs

Netlab is a powerful tool for quickly setting up lab environments. It abstracts the settings of the lab environment, allowing the same labs to be run in various configurations. With Netlab, you can write and use preconfigured network device settings and topologies as code, enabling you to set up and reset labs efficiently.

The BGP labs provided in this setup are prebuilt scenarios designed to teach the basics of EBGP (External Border Gateway Protocol). By leveraging Netlab, you can significantly reduce the overhead of creating a lab environment from scratch. The device settings and topology are already configured, allowing you to focus on the learning objectives of each lab.

## Exploring BGP Labs with GitHub codespaces and Netlab

 Using GitHub Codespaces to run BGP labs using Netlab offers a flexible and efficient way to learn and practice BGP configurations in a simulated environment very quickly.

By automating the setup process and using preconfigured environments, I can focus on developing my skills and understanding BGP concepts without the hassle of manual configuration.

## Setting Up Your Environment

To utilize GitHub Codespaces for deploying the Netlab/BGP labs, follow these steps:

	1.	Launch GitHub Codespaces:
Start an instance connected to the Netlab/BGP labs GitHub repository.
	2.	Start Netlab:
Open a terminal in your Codespace and navigate to the lab directory. Execute the following command to start Netlab:

netlab up

	3.	Connecting to Devices:
Once Netlab is running, all devices in the lab will be running as containers within the instance. Connect to the Linux router and start BGP or any other necessary daemon.
	4.	Using Vtysh:
To configure the router, enter the FRR (Free Range Routing) virtual shell using:

`sudo vtysh`


	5.	Exit and Stop Netlab:
After completing your configurations, exit the router and stop Netlab using:

`netlab down`

Finally, shut down the Codespace instance.

## Skills Acquired

Through this lab experience, I enhanced several skills, including:

	•	Linux system administration
	•	GitHub workflow
	•	Virtualization technologies
	•	Cumulus Linux management
	•	FRR routing daemon configuration
	•	BGP routing techniques

Additional Notes

	•	Cumulus Linux is a distribution designed for networking devices, supporting advanced network protocols such as EVPN, VXLAN, and MLAG. It provides a traditional Linux environment, allowing network engineers to utilize familiar tools like Bash and iptables.
	•	FRR (Free Range Routing) is an open-source routing software suite enabling various network protocols on Linux and Unix-like platforms. It supports protocols like BGP, OSPF, and IS-IS, providing a flexible and efficient routing stack.

Example Commands for FRR Configuration

	1.	To check the version of Cumulus Linux, run:

`cat /etc/os-release`


	2.	To check the status of the FRR service:

`sudo systemctl status frr.service`


	3.	To edit the FRR configuration:

`vi /etc/frr/daemons`

	•	Use i to enter insert mode and make necessary changes, then save and quit with :wq.

	4.	Restart the FRR service to apply changes:

`sudo systemctl restart frr.service`





References:

https://bgplabs.net/