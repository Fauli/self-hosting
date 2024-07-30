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

| Interface    | Connected | VLAN | Bridge     |
| ------------ | --------- | ---- | ---------- |
| Interface 1  | Personal  | 10   | pc-bridge  |
| Interface 2  | Personal  | 10   | pc-bridge  |
| Interface 3  | Personal  | 10   | pc-bridge  |
| Interface 4  | Personal  | 10   | pc-bridge  |
| Interface 5  | Personal  | 10   | pc-bridge  |
| Interface 6  | Personal  | 10   | pc-bridge  |
| Interface 7  | Cluster   | 10   | k8s-bridge |
| Interface 8  | Cluster   | 20   | k8s-bridge |
| Interface 9  | Cluster   | 20   | k8s-bridge |
| Interface 10 | Cluster   | 20   | k8s-bridge |

## Logical network diagram

![Network Diagram](.attachements/Network-Diagram.png)

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

## Steps to setup the configuration
```bash
## bridges connect the required ports, so no VLAN tagging occurs
/interface bridge
add name=pc-bridge
add name=k8s-bridge

/interface/list/member
add list=LAN interface=pc-bridge
add list=LAN interface=k8s-bridge

# give the attached clients IPs using DHCP
/ip address
add address=192.168.10.1/24 interface=pc-bridge
add address=10.13.37.1/24 interface=k8s-bridge

/ip pool
add name=pool10 ranges=192.168.10.10-192.168.10.100
add name=pool20 ranges=10.13.37.10-10.13.37.100

/ip dhcp-server
add name=dhcp10 interface=pc-bridge address-pool=pool10 disabled=no
add name=dhcp20 interface=k8s-bridge address-pool=pool20 disabled=no

/ip dhcp-server network
add address=192.168.10.0/24 gateway=192.168.10.1 
add address=10.13.37.0/24 gateway=10.13.37.1 

## setup firewall rules
/ip firewall filter
add chain=forward action=accept in-interface=pc-bridge out-interface=k8s-bridge protocol=tcp dst-port=8006 # proxmos interface
add chain=forward action=accept in-interface=pc-bridge out-interface=k8s-bridge protocol=tcp dst-port=22 # SSH access
add chain=forward action=accept in-interface=pc-bridge out-interface=k8s-bridge protocol=tcp dst-port=6443 # k8s api server
add chain=forward action=accept in-interface=pc-bridge out-interface=k8s-bridge protocol=tcp dst-port=30000-32767 # exposed services
add chain=forward action=accept in-interface=pc-bridge out-interface=k8s-bridge protocol=tcp dst-port=2049 # NFSv4
add chain=forward action=drop in-interface=pc-bridge out-interface=k8s-bridge  # drop the rest
add chain=forward action=drop in-interface=k8s-bridge out-interface=pc-bridge  # drop the rest

## map the ports to the actual bridges
/interface bridge port
set [find interface=ether1] bridge=pc-bridge
set [find interface=ether2] bridge=pc-bridge
set [find interface=ether3] bridge=pc-bridge
set [find interface=ether4] bridge=pc-bridge
set [find interface=ether5] bridge=pc-bridge
set [find interface=ether6] bridge=pc-bridge
set [find interface=ether7] bridge=k8s-bridge
set [find interface=ether8] bridge=k8s-bridge
set [find interface=ether9] bridge=k8s-bridge
set [find interface=ether10] bridge=k8s-bridge
```

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
| Metrics | Grafana & Prometheus |   |
| Security validation | kube-bench |  |
| Admin | kubernetes dashboard | | 

# Additional Thoughts

## Make wireguard great again

```bash
/interface wireguard
add name=wireguard0 listen-port=13337 private-key="HIDDEN_DATA"

/ip address
add address=192.168.99.1/24 interface=wireguard0

/interface wireguard peers
add public-key="peerPublicKey" allowed-address=192.168.99.2/32 interface=wireguard0

/ip route
add dst-address=192.168.10.0/24 gateway=192.168.99.1
add dst-address=10.13.37.0/24 gateway=192.168.99.1

/ip firewall filter
add chain=input action=accept protocol=udp port=51820
add chain=forward action=accept in-interface=wireguard0 out-interface=VLAN10
add chain=forward action=accept in-interface=wireguard0 out-interface=VLAN20

/ip firewall nat
add action=masquerade chain=srcnat out-interface=wireguard0
```

## Image registry on the NAS

Is that smart? Where else?

## Use cilium to protect network further

Do it

## Gateway API

Maybe worth from the start?

## Cloudflare

Or some other service?

## PXE Boot

Makes life easier maybe?
