---
title: "Setup GCP Containerlab host with VPN to PLAB"
date: 2025-03-010T01:00:00
tags:
  - containerlab
  - gcp
  - plab

categories: 
- networking

draft: false
url: "GCP-to-PLAB"
summary: "Setup process of GCP host vm running containerlab, with access to internal PLAB over VPN"
description: "Setup process of GCP host vm running containerlab, with access to internal PLAB over VPN"
---

This describes the process of setting up a new GCP account, using a temporary email, to make use of the GCP 90 day $300 free trial.  Will use GCP to run a containerlab host that I can start/stop as needed and also scale up or down depending on lab parameters. But want to be able to access the host and the lab nodes from my PLAB local host development environment.


# 1. Setup new GMAIL account
create gmail account for for the GCP free tier lab

used false dob (1/1/80), used phone as second, send 
changed data and privacy setting to turn of all collecting and sharing possible
enabled 2-factor authentication
changed profile picture - makes it easier to see what connecting to

Next steps..
- [x] Deleted and erase my previous Google GCP  old account - 

open up GCP 

Need to enable GCP console and add Credit Card (used virtual card on captial one)
Created new Project call "plab"
activate and authorise Cloud Shell
set the plab project to be default project `gcloud config set project plab`

# 2. Prepare GCP VM to run containerlab 

ref: https://github.com/infotechca/gns3-on-gcp

`gcloud compute images list`

NAME: ubuntu-2204-jammy-v20250305
PROJECT: ubuntu-os-cloud
FAMILY: ubuntu-2204-lts
DEPRECATED: 
STATUS: READY

directions say to use a public boot disk image of ubunut, then create a disk image with license to allow nested hosts.  This is key because to run VM, on a VM, need to be enabled.

`gcloud compute disks create disk1 --image-project ubuntu-os-cloud --image-family ubuntu-2204-lts --zone us-east4-a`

```
# create image
gcloud compute images create clab-vm-image \
  --source-disk disk1 --source-disk-zone us-east4-a \
  --licenses "https://www.googleapis.com/compute/v1/projects/vm-options/global/licenses/enable-vmx"

# create vm
gcloud compute instances create clab-vm --zone us-east4-a \
              --min-cpu-platform "Intel Skylake" --machine-type=n1-standard-8 \
              --boot-disk-size=30GB --boot-disk-type=pd-ssd \
              --tags http-server,https-server \
              --image clab-vm-image
```
**NOTE: I actually created clab-vm from GUI, not with instance create cli shown above.  Basic settings:
	- n1-standard-8.  30Gb 
	- boot disk image = clab-vm-image.  
	- Make sure to enabled CanIPForward - This is key, because it allows you later to route subnets (for example the container lab docker subnet, to the vm instance.  This is best set at instance create, but can be added later too. )
started VM

Modify GCP Firewall rules for the clab-vm instance (different than policies)
- block icmp, ssh and others from source all
- allow ssh and icmp from plab wan **204.111.113.247**

Added my mac ssh key for user account 

confirm vm nesting is setup correctly
`grep -cw vmx /proc/cpuinfo`
 - should be number higher than 1


Check for updates and prep Ubuntu
sudo apt update && sudo apt upgrade -y
	- update package list and installed packages
sudo dpkg-reconfigure unattended-upgrades
	- confirm automatic security updates are enabled

## 3. install containerlab
used the script and curl entry on containerlab page
mkdir containerlab
Followed instructions on containerlab website. 

Reference: Link to {{< external-link "https://containerlab.dev/install/" >}}containerlab{{< /external-link >}} Website.


downloaded the image file for srl using `docker pull image` command
then was able to deploy my test topology

## 4. setup vpn tunnel
 in GCP go to Network connectivity -> VPN.  Create VPN connection (classic)
Create cloud vpn gw with GCP public ip and and Tunnel to my PLAB Firewall public ip over IPSECv2
Need to get a public IP in the GCP for the Cloud VPN
adjust FGT to point VPN tunnel to new IP 
added firewall rule to allow plab prod network (192.168.33.0) to have ingress access to VM instances on all ports
added route to allow the clab subnet (172.20.0.0/16) to be routed the clab-vm instance


## 5. setup containerlab to support Aruba-CX OS VM image ()
Ensure that the VM that containerlab runs on has [Nested virtualization enabled](https://stafwag.github.io/blog/blog/2018/06/04/nested-virtualization-in-kvm/) to support vrnetlab-based containers.
clone the github repo for hellt/vrnetlab ` git clone https://github.com/hellt/vrnetlab && cd vrnetlab`
in the folder for each suppored nos vm image - find a readme with instructions on how to install.
In case of Aruba CX OS - get the ova file and extract(untar) to the vmdk image
	`tar -xvf your_ova_file.ova`
copy the file to GCP clab-vm using scp
move the file to the vrnet/aoscx directory and follow direction in the vrnet readme to make the file into docker image
confirm docker image is avaiable with the `docker images` command
then you can refer to this node in topo file for example 
```
kind: aruba_aoscx
image: vrnetlab/aruba_arubaos-cx:10.13.0005`
# where the :10.13.0005 is custom tag created in this example
```
