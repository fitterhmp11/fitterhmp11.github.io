
---
title: "Home Lab Progress: Containerlab Configuration Management with SCP Server"
date: 2025-10-10
draft: false
tags: ["homelab", "containerlab", "docker", "aruba-cx", "networking", "devops"]
url: "home-lab-scp-server"
summary: "Home Lab Progress: Containerlab Configuration Management with SCP Server"
description: "Home Lab Progress: Containerlab Configuration Management with SCP Server"
---

## Overview

This week I made significant progress on my home lab setup, focusing on building a sustainable configuration management workflow for my Containerlab topology using Aruba CX switches. The key achievement was implementing an SCP server in a Docker container to handle network device configurations.

## The Configuration Challenge

I initially set up Containerlab with a simple Aruba CX topology. The config files in the Containerlab folders provide basic startup configurations—enabling interfaces and setting up basic static routing. However, there's a fundamental limitation with these startup scripts:

- They execute as scripts at container start, not as configuration loads
- The configuration commands must be in the correct order
- You cannot simply load a saved configuration file

This approach works for initial setup but doesn't support my goal of uploading pre-configured configs or saving and reusing topologies easily.

## Solution: Containerized SCP Server

To solve this, I decided to run an SSH/SCP server on the Linux host as a container. This server can:

- Manage node configuration files
- Integrate with Containerlab networks
- Provide flexibility for future features

### Building the Docker Image

I created a Dockerfile and startup script (with some help from ChatGPT) that builds a Docker image running a Linux client with SSH/SCP server capabilities. I saved these files to a tools directory for future editing and reuse.

### Configuration Details

The setup process revealed several important considerations:

**Authentication:** I initially configured SSH key-based authentication, but discovered that Aruba CX nodes don't easily support SSH keys for SCP—they require username/password authentication. I rebuilt the image with a local user account to accommodate this.

**Networking:** Aruba CX nodes in vrnetlab containers need to use the management VRF to access resources outside the lab. The switches themselves don't have direct knowledge of the Containerlab Docker network. To copy the running configuration via SCP, I had to target the host IP on port 2222 using the `vrf mgmt` context.

**Port Mapping:** The SCP server runs on port 22 inside the container but maps to port 2222 on the local host.

**Data Persistence:** I mounted the SSH key and config file directory to the local host, ensuring data persists across container restarts.

## Success

After working through these challenges, I successfully:

- Started the SCP server container
- Launched my Containerlab topology
- Copied running configurations from Aruba CX switches to the SCP server

## Key Learnings

### Docker Concepts

Working through this project clarified several Docker fundamentals:

- **Dockerfile** contains instructions to create a Docker image and runs during the build process
- **Build scripts** can reference additional scripts that execute once during image creation
- **Runtime commands** and volume mounts configure the container when you run the image
- **Container vs. Image**: Closing or removing a container doesn't affect the underlying image
- **Data persistence** requires mounting volumes to the local host; otherwise data is lost when containers stop

**Useful Docker Commands:**
```bash
docker restart scp-server  # Restarts the container instance
docker stop scp-server     # Stops the running container
docker rm scp-server       # Removes the container instance (not the image)
```

### Linux Skills

I became more familiar with essential Linux commands: `rm`, `cp`, `mv`, `ls -al`

I also generated an SSH key on my Linux host using `ssh-keygen` with the Ed25519 algorithm, which is based on elliptic curve cryptography and is a common modern standard.

### GitHub Repository Structure

I planned and organized a repository directory structure for version control and portability:

- Designed to support Containerlab, Ansible, and other automation tools
- Emphasizing containerization where possible
- Moved my public SSH key to the repo for device access
- Added the secrets folder to `.gitignore` at the root directory to protect sensitive data

## Next Steps

With this foundation in place, I now have a flexible system for managing network device configurations. Future work will focus on expanding the automation capabilities and integrating additional tools into the workflow.