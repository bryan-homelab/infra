# Homelab Infra

Personal homelab: docs for how it works, plus Ansible to configure the three Ubuntu PCs.

Kubernetes is planned but **not installed** yet.

## What’s in the lab

- **3×** Lenovo ThinkCentre M720q (8 GB RAM / 256 GB each)
- **Router:** TP-Link ER605 (separates the lab from home Wi‑Fi; also runs WireGuard)
- **Networks:** home `10.0.0.0/24` · lab LAN `192.168.0.0/24` · WireGuard `10.10.10.0/24`
- **Automation:** Ansible on a Mac, talking to nodes with reserved DHCP addresses

## Which doc should I open?

| I want to know… | Open |
| --- | --- |
| What hardware exists | [Hardware overview](docs/hardware/overview.md) |
| How networking / WireGuard works | [Networking overview](docs/networking/overview.md) |
| How to run Ansible and what it manages | [Ansible overview](docs/ansible/overview.md) |
| What we built over time (problems + decisions) | [Project log](docs/project-log/) |
| Deep networking build notes from early setup | [Networking learnings](docs/networking/learnings.md) |

**Rule of thumb:** current-state how-to lives under `docs/hardware`, `docs/networking`, and `docs/ansible`. The project log is the story of building it. Git is the exact file history.

## Repo layout

```text
ansible/     inventory, ansible.cfg, playbooks
docs/        how-it-works docs + monthly project log
```

## Status

Baseline packages are managed with Ansible on all three nodes. Next up: prepare the hosts for kubeadm / Kubernetes.
