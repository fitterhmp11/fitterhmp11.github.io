---
title: "Docker Network Types Differentiated"
date: 2025-02-19T01:00:00
tags:
  - docker
categories: 
 - networking
 - virtualization

draft: false
url: "docker-network-types"
summary: "Explore the different Docker container network types."
description: "Explore the different Docker container network types."
---


*The source of this post is my own notes taken while watching a NetworkChuck youtube video and then expanded by ChatGPT.*

# Docker Network Types

## 1. Default Bridge Network

The **default bridge network** is created automatically when Docker is installed. It uses the `docker0` virtual bridge interface, which acts as a switch connecting containers.

### Key Features:
- Containers connected to the bridge can communicate with each other.
- The default bridge provides **DHCP**, assigning IPs to containers.
- The host **cannot** access containers unless ports are explicitly exposed.
- Uses the host’s DNS settings (`/etc/resolv.conf`).

### Commands:
- List Docker networks:
  ```sh
  sudo docker network ls
  ```
- Inspect the default bridge network:
  ```sh
  sudo docker inspect bridge
  ```
- Expose a container port to the host:
  ```sh
  sudo docker run -itd --rm -p 80:80 --name web nginx
  ```

## 2. User-Defined Bridge Network

A **user-defined bridge** network is a custom bridge network that provides **better isolation** and **container name resolution**.

### Key Features:
- **Custom network isolation** (does not interact with the default bridge).
- **Automatic DNS resolution** between containers on the same network.
- Recommended over using the default bridge.

### Commands:
- Create a user-defined bridge network:
  ```sh
  sudo docker network create asgard
  ```
- Start a container in the custom network:
  ```sh
  sudo docker run -itd --rm --network asgard --name loki busybox
  ```
- Inspect the network:
  ```sh
  sudo docker inspect asgard
  ```

## 3. Host Network

A container using the **host network** shares the host’s network stack, meaning:
- No subnet, no virtual interfaces – it **directly uses the host's network**.
- No port forwarding required.
- Best for performance-sensitive applications like VPNs (e.g., WireGuard).

### Commands:
- Run a container in the host network:
  ```sh
  sudo docker run -itd --rm --network host --name nginx01 nginx
  ```

## 4. MACVLAN Network

MACVLAN allows **containers to appear as physical devices** on the network, each with its **own MAC address and IP**.

### Key Features:
- Containers behave as if they are separate physical devices on the network.
- Requires enabling **promiscuous mode** on the host interface.
- DHCP is **not** available; static IP assignment is required.

### Commands:
- Create a MACVLAN network:
  ```sh
  sudo docker network create -d macvlan     --subnet 192.168.33.0/24     --gateway 192.168.33.1     -o parent=en1     macvlan-plab
  ```
- Start a container in the MACVLAN network:
  ```sh
  sudo docker run -itd --rm --network macvlan-plab --ip 192.168.33.25 --name busybox01 busybox
  ```

## 5. IPVLAN Network

IPVLAN is similar to MACVLAN but allows containers to share the host’s **MAC address** while getting **unique IPs**.

### Modes:
- **L2 Mode**: Containers share the same subnet as the host.
- **L3 Mode**: Containers use routed networking (requires static routes).

### Commands:
- Create an **L2** IPVLAN network:
  ```sh
  sudo docker network create -d ipvlan     --subnet 192.168.7.0/24     --gateway 192.168.7.1     -o parent=enp0s3 netname
  ```
- Create an **L3** IPVLAN network:
  ```sh
  sudo docker network create -d ipvlan     --subnet 192.168.94.0/24     -o parent=enp0s3 -o ipvlan_mode=l3     --subnet 192.168.95.0/24 newl3network
  ```

## 6. Overlay Network

Used in **Docker Swarm**, the **overlay network** allows containers across multiple hosts to communicate.

### Key Features:
- Abstracts inter-host communication complexity.
- Uses **VXLAN tunneling** to connect containers across hosts.

### Commands:
- Create an overlay network:
  ```sh
  sudo docker network create -d overlay my_overlay
  ```

## 7. None Network

A container with `--network none` has **no network access**.

### Commands:
- Run a container with no network:
  ```sh
  sudo docker run -itd --rm --network none --name isolated busybox
  ```


