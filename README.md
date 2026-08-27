# Homelab Infra

Infrastructure and documentation for my personal homelab.

The repo holds operational docs and Ansible configuration for managing the lab hosts. Kubernetes is planned next but is not installed yet.

## Lab at a glance

- **Compute:** 3× Lenovo ThinkCentre M720q (8 GB RAM / 256 GB each)
- **Router:** TP-Link ER605 (homelab boundary + WireGuard management endpoint)
- **Networks:** home underlay (`10.0.0.0/24`), private lab LAN (`192.168.0.0/24`), WireGuard management (`10.10.10.0/24`)
- **Automation:** Ansible from a Mac control node; nodes addressed via DHCP reservations

## Documentation philosophy

| Source | Answers |
| --- | --- |
| [Technical docs](docs/) | How the homelab works **right now** |
| [Project log](docs/project-log/) | How it was **built over time** (problems, fixes, decisions) |
| Git history | Exact **code/config** changes |

Keep current-state detail in the technical docs. Keep narrative and troubleshooting history in the project log.

## Documentation

| Doc | Description |
| --- | --- |
| [Hardware overview](docs/hardware/overview.md) | Nodes, networking gear, storage, and capacity |
| [Networking overview](docs/networking/overview.md) | Topology, addressing, routing, and WireGuard |
| [Networking learnings](docs/networking/learnings.md) | Earlier build notes for the lab network |
| [Ansible overview](docs/ansible/overview.md) | Inventory, SSH, privilege escalation, playbooks |
| [Project log](docs/project-log/) | Monthly build / learning journal |

## Layout

```text
ansible/          # inventory, ansible.cfg, playbooks
docs/             # current-state docs + project log
```

## Status

Ansible manages baseline package configuration on all three nodes. Next: prepare hosts for kubeadm/Kubernetes.
