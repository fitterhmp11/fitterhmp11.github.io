---
title: "GCP to Fortigate Lab VPN Tunnel"
date: 2025-02-11T01:00:00
tags:
  - gcp
  - fortinet
  - ssh
  - macos
categories: 
 - networking
 - cloud
 
draft: false
url: "gcp-to-fgt-vpn"
summary: "Setting up a VPN tunnel between GCP Lab environment and Fortigate Lab environment"
description: "Basic Git concepts, terms and commands"

---

# Setting up GCP to FortiGate VPN

My current VM, `clab-vm`, has the IP address `10.150.0.4`. 

In the details, under the "Network Interfaces" section, the "Network" field shows that it is in the default VPC. In this case, the subnet is part of the `us-east4` region with the CIDR `10.150.0.0/20`.

I could not use an ephemeral IP on the GCP side of the VPN. According to Gemini AI:

> Ephemeral IPs are designed for temporary use, and their address might change unexpectedly, breaking your VPN connection. While they can reduce costs in some situations, the instability caused by the changing IP address will likely outweigh any cost savings if you need consistent connectivity to your home network. You’ll constantly need to update the FortiGate configuration and risk interruptions in service. A static IP, despite the added cost, provides the reliable, consistent connection necessary for a home VPN.

So, I had to reserve a public IP for the GCP VPN Classic tunnel, which will increase cost.

When I tried to set the remote peer (my home FortiGate), I couldn’t use my Dynamic DNS (DDNS) name and had to use the static IP instead.  If my public IP changes in the future, I will need to update the GCP tunnel configuration.

The tunnel came up once I created the FortiGate policies and routes. Unfortunately, GCP does not allow DNS names in the tunnel configuration, so I need to figure out how to run a script to bring the tunnel and VM up and down automatically.

Next, I had to add a firewall rule to the GCP VM to allow my private network to access the GCP instances. I opened all ports for this purpose. Afterward, I could ping my GCP VM.

Once the VPN tunnel was up and the GCP VM was started, the tunnel between my FortiGate PLAB prod subnet and the GCP `plab1` network worked as expected.

I then wanted to SSH into the VM. To do so, I needed to ensure that I had a key on my MacBook Pro.

Using ChatGPT for help, I realized that I didn’t have any keys in my `~/.ssh` folder, so I generated one. The `-C` option is for adding a comment (in this case, my email), and the `-f` option specifies where to store the key and its name. I named it after my device, which helps keep track of its purpose. It's best to have a separate key for each device. The `-C` option doesn’t affect the functionality but helps with organization. I didn’t add a passphrase because I didn’t want to be prompted for it every time I logged in with the key.

The command I used to generate the SSH key:

```bash
ssh-keygen -t ed25519 -C name@email.com -f ~/.ssh/id_ed25519_macbookpro
```

I also had to ensure the private key had the proper permissions:

```bash
chmod 600 ~/.ssh/id_ed25519_macbookpro
```

This generated two files with the key name, one of which ends in `.pub`. I opened the `.pub` file and added the key to the SSH keys section of my GCP VM.

Now, I was able to SSH into the GCP VM with the following command:

```bash
ssh -i ~/.ssh/id_ed25519_macbookpro username@10.150.0.4
```

To simplify SSH access from the terminal, I created a Bash script. I edited the `~/.ssh/config` file:

```bash
nano ~/.ssh/config
```

I added the following configuration:

```text
Host gcp_clab-vm
  HostName 10.150.0.4
  User username
  IdentityFile ~/.ssh/id_ed25519_macbookpro
```

Now, I can access my `clab-vm` by simply typing:

```bash
ssh gcp_clab-vm
```

### Future Work Ideas:

- Install the GCP CLI on my MacBook Pro or Mac mini to run GCP CLI commands without needing to use the GCP portal.
- Create a script to automatically update the peer address of the GCP tunnel if my FortiGate's public IP changes.
- Check whether I need to open both UDP/TCP port 500 and 4500 to my FortiGate through the Eero, and update as necessary.
- Create a script to quickly start the `clab-vm` on GCP.
- Create a script to disable the GCP VPN interface on the FortiGate to reduce costs.
- Simplify a script to start the GCP `plab1` network with the tunnel and access.

Additionally, I want to explore accessing ContainerLab running in the GCP VM from my Mac and setting up a topology. I also need to confirm if I can access the routers and switches from my production network.

