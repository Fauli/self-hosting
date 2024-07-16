# Self Hosting

# Table of Contents
- [Introduction](#introduction)
- [Hardware](#hardware)
- [Network Setup](#network-setup)
    - [Logical network diagram](#logical-network-diagram)
    - [Firewall rules](#firewall-rules)
- [Kubernetes](#kubernetes)
    - [Kubernetes Node Setup](#kubernetes-node-setup)
    - [Kubernetes Components](#kubernetes-components)


# Introduction

<img src=".attachements/sloth-hosting.png" width="250"> <!-- Adjust width as needed -->

In order to host services I need, I'm using the here described setup.
This is a living document so far and should only be seen as collection of my ideas 😁

# Hardware

| What    | Model                       | Description                                                       |
| ------- | --------------------------- | ----------------------------------------------------------------- |
| Router  | Mikrotik RB4011iGS+         | Latest firmware, central network switch and router for the setup  |
| Servers | 3x Lenovo ThinkCentre M710s | Intel Core i7-7700 <br />64 GB Memory                             |


# Network Setup

In order to separate the internal personal LAN from the cluster, a new VLAN is created.

The Kubernetes nodes are connected via the physical ports on the router.

| Interface    | Connected | VLAN |
| ------------ | --------- | ---- |
| Interface 1  | Personal  | 10   |
| Interface 2  | Personal  | 10   |
| Interface 3  | Personal  | 10   |
| Interface 4  | Personal  | 10   |
| Interface 5  | Personal  | 10   |
| Interface 6  | Personal  | 10   |
| Interface 7  |           | 10   |
| Interface 8  | Cluster   | 20   |
| Interface 9  | Cluster   | 20   |
| Interface 10 | Cluster   | 20   |


## Logical network diagram

![Network Diagram](.attachements/Network-diagram.png)

## Firewall rules

| Chain   | Source | Destination | Port        | Protocol | Action     | Description                |
| ------- | ------ | ----------- | ----------- | -------- | ---------- | -------------------------- |
| forward | VLAN10 | VLAN20      | 8006        | TCP      | accept     | Proxmox interface          |
| forward | VLAN10 | VLAN20      | 22          | TCP      | accept     | SSH access to nodes        |
| forward | VLAN10 | VLAN20      | 6443        | TCP      | accept     | API Server                 |
| forward | VLAN10 | VLAN20      | 30000-32767 | TCP      | accept     | Access to exposed services |
| forward | VLAN20 | VLAN10      | 2049        | TCP/UDP  | accept     | NFS4 access for CSI driver |
| srcnat  | VLAN10 | WAN         | -           | -        | masquerade | NAT for internet access    |
| srcnat  | VLAN20 | WAN         | -           | -        | masquerade | NAT for internet access    |

# Kubernetes

## Kubernetes Node Setup

| Server | VM  | Node          |
| ------ | --- | ------------- |
| PC1    | VM1 | Master Node 1 |
| PC1    | VM2 | Worker Node 1 |
| PC1    | VM3 | Master Node 2 |
| PC1    | VM4 | Worker Node 2 |
| PC1    | VM5 | Master Node 3 |
| PC1    | VM6 | Worker Node 3 |

## Kubernetes Components

| What    | Service     | Description |
| ------- | ----------- | ----------- |
| Ingress | using nginx |             |


# Additional Thoughts

## Cloudflare

## PXE Boot

''