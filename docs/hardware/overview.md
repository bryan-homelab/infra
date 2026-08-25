# Hardware Overview

## Purpose

This document tracks the physical hardware that makes up the homelab. It is intended to stay current as nodes, networking equipment, storage, and other devices are added or upgraded.

## Compute Nodes

The homelab currently consists of three Lenovo ThinkCentre M720q mini PCs.

| Node | Model | Memory | Internal Storage |
| --- | --- | ---: | ---: |
| Node 1 | Lenovo ThinkCentre M720q | 8 GB RAM | 256 GB |
| Node 2 | Lenovo ThinkCentre M720q | 8 GB RAM | 256 GB |
| Node 3 | Lenovo ThinkCentre M720q | 8 GB RAM | 256 GB |

### Current Cluster Capacity

Across all three nodes:

- **3 compute nodes**
- **24 GB total RAM**
- **768 GB total internal storage**

These figures represent installed physical capacity, not necessarily the amount available to workloads after operating-system and platform overhead.

## Networking Hardware

### TP-Link ER605

The TP-Link ER605 is the homelab router and network boundary.

Its current responsibilities include:

- Connecting the homelab to the upstream home network
- Providing the homelab LAN
- Routing between relevant networks
- Hosting the WireGuard endpoint used for management access

Detailed addressing, routing, and WireGuard configuration are documented in the networking documentation.

### TP-Link 8-Port Gigabit Easy Smart Switch

An 8-port TP-Link Gigabit Easy Smart Switch provides wired connectivity between the homelab devices.

The switch gives the lab room to connect the three compute nodes, router, and additional Ethernet devices as the environment grows.

## Console / Display

### 7-Inch Monitor

A 7-inch monitor is available for local console access.

This is useful when a node cannot be reached remotely, such as during:

- Initial operating-system installation
- Boot troubleshooting
- Network configuration problems
- SSH failures
- Recovery work

Normal administration is performed remotely once networking is available.

## Physical Topology

At a high level:

```text
Upstream Home Network
        |
        |
    TP-Link ER605
        |
        |
  TP-Link 8-Port
 Gigabit Smart Switch
    /      |      \
   /       |       \
Node 1   Node 2   Node 3
M720q    M720q    M720q

7-inch monitor
   |
   +---- connected to a node when local console access is needed
```

## Inventory Summary

| Category | Hardware | Quantity |
| --- | --- | ---: |
| Compute | Lenovo ThinkCentre M720q | 3 |
| Router | TP-Link ER605 | 1 |
| Switching | TP-Link 8-Port Gigabit Easy Smart Switch | 1 |
| Console | 7-inch monitor | 1 |

## Future Updates

Update this document whenever the physical lab changes, including:

- RAM upgrades
- Storage upgrades
- Additional nodes
- Dedicated storage or NAS hardware
- Additional network interfaces
- Switch/router changes
- UPS or power-management hardware
- Other permanent homelab equipment
