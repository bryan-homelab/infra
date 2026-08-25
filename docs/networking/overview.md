# Networking Overview

## Purpose

The homelab network separates lab infrastructure from the upstream home network while still providing secure management access from a personal MacBook.

The ER605 acts as the boundary router for the homelab. Its WAN interface connects to the existing home network, while its LAN interface provides a separate private subnet for the homelab nodes.

## Network Topology

```text
                         Home Network
                         10.0.0.0/24

MacBook -------------------------------- Home Router
  |                                         10.0.0.1
  |                                             |
  |                                      ER605 WAN
  |                                      10.0.0.54
  |                                             |
  |                                      TP-Link ER605
  |                                      LAN: 192.168.0.1
  |                                             |
  |                                      192.168.0.0/24
  |                                             |
  |                                           Switch
  |                                      /      |      \
  |                                  Node 1   Node 2   Node 3
  |
  +========== WireGuard ==========> ER605
       Mac: 10.10.10.2              WG: 10.10.10.1
```

## Networks

| Network | Purpose |
| --- | --- |
| `10.0.0.0/24` | Existing home network / WireGuard underlay |
| `192.168.0.0/24` | Private homelab LAN |
| `10.10.10.0/24` | WireGuard management network |

## ER605 Interfaces

| Interface / Role | Address | Purpose |
| --- | --- | --- |
| WAN | `10.0.0.54` | Connects the ER605 to the home network |
| LAN | `192.168.0.1` | Default gateway for the homelab LAN |
| WireGuard | `10.10.10.1` | ER605 identity on the VPN network |

The ER605 therefore participates in three separate IP networks.

## MacBook Management Access

The MacBook has two relevant network identities while connected to the VPN:

- Its normal Wi-Fi address on `10.0.0.0/24`
- Its WireGuard address, `10.10.10.2`

WireGuard provides access to the otherwise separate `192.168.0.0/24` homelab network.

The configuration uses split tunneling. Homelab traffic is routed through WireGuard, while ordinary Internet traffic continues to use the normal home-network default gateway.

Relevant WireGuard destinations:

```text
192.168.0.0/24 -> WireGuard
10.10.10.1/32  -> WireGuard
```

## WireGuard Transport

The ER605 WireGuard endpoint is:

```text
10.0.0.54:51820/UDP
```

`10.0.0.54` and `10.10.10.1` have different purposes:

- `10.0.0.54` is reachable on the existing physical network and transports WireGuard packets.
- `10.10.10.1` is the ER605's virtual IP identity inside the WireGuard network.

A packet from the MacBook to a homelab node is conceptually transported like this:

```text
Inner packet:
10.10.10.2 -> 192.168.0.x

        |
        | WireGuard encrypts and encapsulates it
        v

Outer packet:
Mac home-network IP -> 10.0.0.54:51820/UDP

        |
        | Home network transports the outer packet
        v

ER605
        |
        | WireGuard decrypts the inner packet
        | ER605 performs normal routing
        v

192.168.0.0/24 homelab LAN
```

The existing home network is the **underlay**. WireGuard provides the secure **overlay** used for management.

## Routing

When the MacBook sends traffic to a homelab address such as `192.168.0.x`, macOS selects its WireGuard `utun` interface because `192.168.0.0/24` is associated with the WireGuard peer.

WireGuard encrypts the packet and transports it to `10.0.0.54:51820` over the MacBook's physical Wi-Fi interface.

After the ER605 decrypts the packet, it sees the original destination in `192.168.0.0/24` and forwards it through its LAN interface.

Return traffic destined for `10.10.10.2` is associated with the MacBook WireGuard peer and sent back through the encrypted tunnel.

## Node Management

All homelab nodes are reachable from the MacBook through WireGuard.

Management traffic follows:

```text
MacBook
   |
   | WireGuard
   v
ER605
   |
   | Layer 3 routing
   v
Homelab LAN
   |
   +-- Node 1
   +-- Node 2
   +-- Node 3
```

This allows direct administration such as SSH without relying on temporary reverse SSH tunnels.

## Security Notes

- WireGuard private keys must never be committed to Git.
- The MacBook WireGuard configuration contains a private key and should remain outside the public repository or be explicitly ignored.
- Public keys are safe to distribute as part of peer configuration.
- WireGuard authenticates and encrypts VPN traffic, but it does not replace firewall policy.
- Outbound/egress controls are a separate security concern from VPN access.

## Verification

Useful commands on macOS:

```bash
# Inspect the route selected for a homelab host
route -n get <homelab-node-ip>

# Inspect WireGuard state and handshake information
sudo wg show

# Test the ER605 WireGuard interface
ping 10.10.10.1

# Test a homelab node
ping <homelab-node-ip>

# Connect to a node
ssh <user>@<homelab-node-ip>
```

A healthy WireGuard peer should show a recent handshake and traffic transferred in both directions.
