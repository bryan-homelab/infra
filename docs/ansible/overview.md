# Ansible

How this lab uses Ansible to configure the three Ubuntu machines.

If you only remember one thing: **you run Ansible from your Mac**, and it SSHs into the three nodes to make sure they look the way the playbooks say they should.

## Big picture

| Role | What it is in this lab |
| --- | --- |
| Control machine | Your Mac — where you run `ansible` / `ansible-playbook` |
| Managed hosts | The three ThinkCentre PCs running Ubuntu 26.04 |
| Inventory | A file that lists those machines and how to reach them |
| Playbook | A checklist of desired settings (packages, etc.) |

Flow:

```text
Your Mac
   │
   │  1. Read inventory (who are the nodes?)
   │  2. SSH in as user "bryan" (key-based login)
   │  3. Run tasks
   │  4. If a task needs admin rights, ask for sudo password
   │     and escalate with sudo.ws
   ▼
node01 (192.168.0.100)
node02 (192.168.0.102)
node03 (192.168.0.103)
```

You never SSH in as `root`. Ansible logs in as `bryan`, then elevates only when a task needs it.

## Before you run anything

1. Be on the VPN / network path that can reach `192.168.0.x` (WireGuard — see the networking docs).
2. `cd` into the `ansible/` folder in this repo.  
   That matters because `ansible.cfg` lives there. If you run commands from somewhere else, Ansible may not pick up the sudo settings.
3. Have the sudo password for `bryan` ready when a playbook uses privilege escalation (`--ask-become-pass`).

## Inventory (the host list)

**File:** `inventory/hosts.ini`

```ini
[homelab]
node01 ansible_host=192.168.0.100
node02 ansible_host=192.168.0.102
node03 ansible_host=192.168.0.103

[homelab:vars]
ansible_user=bryan
```

What this means in plain English:

- `[homelab]` is a **group** — a nickname for “all three lab PCs.”
- `node01` is a friendly name. Playbooks can say “talk to node01” instead of typing the IP every time.
- `ansible_host=...` is the real address Ansible connects to.
- `ansible_user=bryan` means: log in as `bryan` on every host in that group.

Those IPs are **DHCP reservations** on the ER605 router (tied to each machine’s Ethernet MAC). The Linux boxes are not hard-coded with static IPs; the router always hands them the same address.

## How login works

Ansible uses normal SSH with a public key.

- Your Mac holds the private key (never put that in Git).
- Each node has the matching public key for user `bryan`.
- If SSH works by hand (`ssh bryan@192.168.0.100`), Ansible can usually talk to that host too.

## When Ansible needs root (`become` + `sudo.ws`)

Some tasks (like installing packages) need admin rights. In Ansible that is called **become**.

On these Ubuntu 26.04 nodes, the normal `sudo` command is a newer rewrite called **sudo-rs**. Ansible does not get along with its password prompt, so privilege escalation times out.

**Workaround in this lab:** tell Ansible to run traditional sudo instead, which is installed as `sudo.ws`.

**File:** `ansible.cfg`

```ini
[privilege_escalation]
become_exe = sudo.ws
```

You still type the sudo password when prompted. We did **not** turn on passwordless sudo.

## Playbooks we have today

### `playbooks/check-system.yml`

A smoke test. It connects to the nodes, gathers basic system info (“facts”), and prints:

- hostname
- total memory (MB)

Useful to confirm Ansible can talk to the machines without changing anything.

### `playbooks/baseline.yml`

The first real “make the machines look like this” playbook. It installs:

- `git`
- `vim`
- `unzip`
- `tree`
- `net-tools`

It uses apt with `state: present`, which means: **these packages should be installed**. If they already are, Ansible leaves them alone.

That “only change what’s needed” behavior is **idempotency**. Running the same playbook twice is safe; the second run should report no changes if nothing drifted.

## Commands to use

Always from the `ansible/` directory:

```bash
# Can Ansible see the inventory?
ansible-inventory -i inventory/hosts.ini --list

# Can Ansible reach every node over SSH?
ansible homelab -i inventory/hosts.ini -m ping

# Read-only check (hostname + memory)
ansible-playbook -i inventory/hosts.ini playbooks/check-system.yml

# Install / verify baseline packages (will ask for sudo password)
ansible-playbook -i inventory/hosts.ini playbooks/baseline.yml --ask-become-pass
```

Note: Ansible’s `ping` module is **not** ICMP ping. It means “SSH in, run a tiny Python check, get a successful reply.”

## What’s next

Kubernetes is **not** installed yet. The planned direction is **kubeadm**, and the next Ansible work is preparing these nodes for that.
