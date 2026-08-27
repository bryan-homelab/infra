# Ansible Overview

## Purpose

Ansible provides host-level configuration management for the homelab. The Mac is the control machine. The three physical Ubuntu 26.04 nodes are managed hosts.

```text
Mac (control node)
 │
 │  Ansible over SSH
 │  public-key auth as bryan
 ▼
bryan on each node
 │
 │  become via sudo.ws when required
 ▼
root privileges for privileged tasks
 │
 ├── node01 (192.168.0.100)
 ├── node02 (192.168.0.102)
 └── node03 (192.168.0.103)
```

SSH connects as `bryan`, not as root. Privilege escalation is used only for tasks that need it.

## Inventory

File: `ansible/inventory/hosts.ini`

```ini
[homelab]
node01 ansible_host=192.168.0.100
node02 ansible_host=192.168.0.102
node03 ansible_host=192.168.0.103

[homelab:vars]
ansible_user=bryan
```

| Concept | Role here |
| --- | --- |
| Inventory | List of managed hosts and connection settings |
| Host alias (`node01`, …) | Logical name used in playbooks and ad-hoc commands |
| `ansible_host` | Reachable IP for that alias |
| Group `homelab` | Target for all three lab nodes |
| Group vars | Shared settings for the group (`ansible_user=bryan`) |

Node addresses are stable via ER605 DHCP reservations (not static IPs configured on the Linux hosts). See networking docs for the broader LAN design.

## SSH authentication

Ansible connects with SSH public-key authentication as `bryan`. Private keys stay on the Mac and are not stored in this repository.

## Privilege escalation

File: `ansible/ansible.cfg`

```ini
[privilege_escalation]
become_exe = sudo.ws
```

On these Ubuntu 26.04 nodes, the default `sudo` command is **sudo-rs**. Ansible’s become prompt handling does not work reliably with that implementation. **`sudo.ws`** is the traditional sudo binary and is what Ansible uses for `become`.

Because `ansible.cfg` lives under `ansible/`, run Ansible commands from that directory so the config is discovered.

## Playbooks

| Playbook | Purpose |
| --- | --- |
| `playbooks/check-system.yml` | Gather facts and print hostname / memory (`ansible_facts.*`) |
| `playbooks/baseline.yml` | Ensure baseline packages are installed (`become: true`) |

### Baseline packages

Managed by `baseline.yml` with `ansible.builtin.apt` / `state: present`:

- `git`
- `vim`
- `unzip`
- `tree`
- `net-tools`

### Desired state and idempotency

Playbooks declare the desired end state. Ansible compares that to the host and only makes changes when needed. Rerunning a playbook against hosts that already match should report no changes.

## Running Ansible

From the `ansible/` directory:

```bash
# Inventory / connectivity
ansible-inventory -i inventory/hosts.ini --list
ansible homelab -i inventory/hosts.ini -m ping

# Playbooks
ansible-playbook -i inventory/hosts.ini playbooks/check-system.yml
ansible-playbook -i inventory/hosts.ini playbooks/baseline.yml --ask-become-pass
```

`--ask-become-pass` is required for privilege escalation with the current sudo setup.
