# Homelab Infra

Infrastructure and documentation for my personal homelab.

Today this repo mostly holds design and operational docs. Over time it will also hold automation (Ansible and related tooling) for provisioning and managing the lab.

## Lab at a glance

- **Compute:** 3× Lenovo ThinkCentre M720q (8 GB RAM / 256 GB each)
- **Router:** TP-Link ER605 (homelab boundary + WireGuard management endpoint)
- **Networks:** home underlay (`10.0.0.0/24`), private lab LAN (`192.168.0.0/24`), WireGuard management (`10.10.10.0/24`)

## Documentation

| Doc | Description |
| --- | --- |
| [Hardware overview](docs/hardware/overview.md) | Nodes, networking gear, storage, and capacity |
| [Networking overview](docs/networking/overview.md) | Topology, addressing, routing, and WireGuard |
| [Networking learnings](docs/networking/learnings.md) | Build notes and decisions from setting up the lab network |
| [Architecture](docs/architecture.md) | High-level platform design *(placeholder)* |

## Planned layout

As automation lands, expect something like:

```text
ansible/          # playbooks, roles, inventory
docs/             # design & ops documentation
```

Exact structure may change as the stack settles.

## Status

Early stage — docs first, automation next.
