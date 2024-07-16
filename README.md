# self-hosting


## Introduction

In order to host services I need, I'm using the here described setup.
This is a living document so far and should only be seen as collection of my ideas 😁

## Hardware

| What    | Model                       | Description                                                       |
| ------- | --------------------------- | ----------------------------------------------------------------- |
| Router  | Mikrotik RB4011iGS+         | Latest firmware, central network switch and router for the setup  |
| Servers | 3x Lenovo ThinkCentre M710s | Intel Core i7-7700 <br />64 GB Memory                             |

## Network Setup

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

### Logical network diagram
![Network Diagram](.attachements/Network-diagram.png)

### Firewall rules

| Chain   | Source | Destination | Port        | Protocol | Action     | Description                |
| ------- | ------ | ----------- | ----------- | -------- | ---------- | -------------------------- |
| forward | VLAN10 | VLAN20      | 8006        | TCP      | accept     | Proxmox interface          |
| forward | VLAN10 | VLAN20      | 22          | TCP      | accept     | SSH access to nodes        |
| forward | VLAN10 | VLAN20      | 6443        | TCP      | accept     | API Server                 |
| forward | VLAN10 | VLAN20      | 30000-32767 | TCP      | accept     | Access to exposed services |
| forward | VLAN20 | VLAN10      | 2049        | TCP/UDP  | accept     | NFS4 access for CSI driver |
| srcnat  | VLAN10 | WAN         | -           | -        | masquerade | NAT for internet access    |
| srcnat  | VLAN20 | WAN         | -           | -        | masquerade | NAT for internet access    |

## Kubernetes Components

| What    | Service     | Description |
| ------- | ----------- | ----------- |
| Ingress | using nginx |             |
