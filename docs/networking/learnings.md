# Networking Build Notes & Learnings

## Goal

The goal was to create a separate homelab network behind an ER605 while still allowing a MacBook on the existing home network to securely manage the lab nodes.

The final design uses:

- `10.0.0.0/24` as the existing home network
- `192.168.0.0/24` as the private homelab network
- `10.10.10.0/24` as a WireGuard management network
- The ER605 as the router between the VPN and homelab LAN

## 1. Building a Separate Homelab Network

The ER605 was connected to the existing home network through its WAN side.

Its WAN address is:

```text
10.0.0.54
```

The ER605 provides a separate LAN:

```text
192.168.0.0/24
```

with the ER605 at:

```text
192.168.0.1
```

This creates a routing boundary between the existing home network and the lab.

### What I learned

An IP address only makes sense together with its subnet.

A host first checks whether a destination belongs to one of its directly connected networks. If it does not, the host sends the packet toward an appropriate route, often the default gateway.

For the MacBook, a destination in `192.168.0.0/24` is not part of its normal home subnet, so without a more specific route it falls back to the home router.

## 2. Routing and the Default Gateway

Before WireGuard was configured, checking the route to the lab demonstrated how macOS makes routing decisions.

Useful command:

```bash
route -n get <destination>
```

When no specific homelab route existed, traffic for the lab used the normal default gateway:

```text
default -> 10.0.0.1
```

### Key lesson

The default gateway is not necessarily the final destination. It is the next router a host uses when it does not have a more specific route.

Routers then make their own routing decisions.

More-specific routes take precedence over the default route.

## 3. Temporary Static Routing

During setup, a temporary route was used to direct homelab traffic toward the ER605 WAN address.

Conceptually:

```text
192.168.0.0/24 -> 10.0.0.54
```

This helped demonstrate routing but later conflicted with the route WireGuard needed to install.

The conflict appeared when `wg-quick` reported:

```text
route: writing to routing socket: File exists
```

Checking the route showed that `192.168.0.0/24` was still using:

```text
gateway: 10.0.0.54
interface: en0
```

The old static route was removed so WireGuard could own the homelab route.

### What I learned

When a networking tool reports that a route already exists, inspect the routing table instead of blindly adding more routes.

Useful command:

```bash
route -n get <destination>
```

## 4. Why WireGuard Was Added

The upstream home router did not have a route into the private homelab network, and changing the upstream router was not an available option.

WireGuard provided another solution: create a secure virtual network between the MacBook and ER605.

WireGuard addresses:

```text
ER605:   10.10.10.1
MacBook: 10.10.10.2
```

The ER605 listens for WireGuard traffic at:

```text
10.0.0.54:51820/UDP
```

## 5. WireGuard Keys and Peers

Each WireGuard peer has a private/public key pair.

The private key remains on the device that owns it.

The public key is given to the peer.

Conceptually:

```text
Mac private key     -> stays on Mac
Mac public key      -> configured on ER605

ER605 private key   -> stays on ER605
ER605 public key    -> configured on Mac
```

This allows each side to authenticate the other cryptographically.

### Important

Never commit WireGuard private keys to Git.

## 6. Mac WireGuard Configuration

The Mac configuration followed this general structure:

```ini
[Interface]
PrivateKey = <MAC_PRIVATE_KEY>
Address = 10.10.10.2/32

[Peer]
PublicKey = <ER605_PUBLIC_KEY>
Endpoint = 10.0.0.54:51820
AllowedIPs = 192.168.0.0/24, 10.10.10.1/32
PersistentKeepalive = 25
```

### What each field means

**`Address`**

The Mac's virtual identity on the WireGuard network.

**`PublicKey`**

The identity of the ER605 peer.

**`Endpoint`**

Where the encrypted WireGuard packets must be physically delivered.

The endpoint uses `10.0.0.54`, not `10.10.10.1`, because the tunnel needs an address reachable through the network that already exists.

**`AllowedIPs`**

Identifies traffic associated with this peer. With `wg-quick`, these destinations are also used to install routes.

For this setup:

```text
192.168.0.0/24
10.10.10.1/32
```

go through WireGuard.

**`PersistentKeepalive`**

Periodically sends authenticated traffic to help keep NAT/firewall state alive.

## 7. Bringing Up WireGuard

WireGuard tools were installed on macOS.

The tunnel was started with:

```bash
sudo wg-quick up ~/lab-wg.conf
```

The output showed macOS creating a virtual interface such as:

```text
utun6
```

The exact number is assigned by macOS and can change between sessions.

The tunnel can be stopped with:

```bash
sudo wg-quick down ~/lab-wg.conf
```

### What I learned about `utun`

The Mac now has multiple networking paths.

Conceptually:

```text
en0
└── physical Wi-Fi
    └── home-network IP

utunX
└── virtual tunnel interface
    └── 10.10.10.2
```

Traffic to the homelab is handed to the `utun` interface.

WireGuard encrypts and encapsulates that traffic, and the resulting outer packets leave through the physical Wi-Fi interface.

## 8. Inner vs. Outer Packets

This was one of the most important concepts from the setup.

Suppose the Mac sends traffic to a homelab node.

The packet the application actually wants to send is the **inner packet**:

```text
Source:      10.10.10.2
Destination: 192.168.0.x
```

WireGuard encrypts it and transports it inside an **outer packet**:

```text
Source:      Mac home-network IP
Destination: 10.0.0.54
Protocol:    UDP
Port:        51820
```

Conceptually:

```text
Outer IP packet
└── UDP
    └── WireGuard encrypted payload
        └── Inner IP packet
            └── application traffic
```

### Why this is called a tunnel

The actual packet is encapsulated inside traffic that the existing underlying network already knows how to transport.

The home network only needs to know how to reach `10.0.0.54`.

It does not need to understand the inner `10.10.10.0/24` WireGuard network or the final homelab destination.

## 9. Normal Routing vs. Tunneling

Normal routing does not usually wrap an IP packet inside another IP packet.

During ordinary routed Ethernet traffic:

- The IP destination generally remains the final destination.
- Layer 2/MAC addressing changes as the packet moves between Layer 2 networks.
- Each router examines the IP destination and chooses the next hop.

With WireGuard:

```text
inner IP packet
        |
        | encryption + encapsulation
        v
outer IP packet
```

The outer packet transports the encrypted inner packet between tunnel endpoints.

## 10. Underlay vs. Overlay

The physical/existing network is the **underlay**:

```text
Mac home-network IP -> 10.0.0.54
```

WireGuard creates an **overlay**:

```text
10.10.10.2 <-> 10.10.10.1
```

The overlay depends on the underlay for transport.

This is related to the same general tunneling/encapsulation idea used by technologies such as VXLAN, although WireGuard and VXLAN solve different problems.

## 11. WireGuard IPs as Virtual Identities

The ER605 has several identities because it participates in several networks:

```text
10.0.0.54     -> home/WAN network
192.168.0.1   -> homelab LAN
10.10.10.1    -> WireGuard network
```

Likewise, the Mac has its normal home-network identity and:

```text
10.10.10.2 -> WireGuard identity
```

`10.10.10.1` is not an intermediate destination that every homelab packet must be addressed to.

If the Mac sends traffic directly to the ER605's WireGuard interface, the destination is `10.10.10.1`.

If the Mac sends traffic to a node, the inner packet remains addressed to that node, and the ER605 routes it after WireGuard decrypts it.

## 12. Verifying the Tunnel

WireGuard state can be inspected with:

```bash
sudo wg show
```

A successful setup showed:

```text
endpoint: 10.0.0.54:51820
allowed ips: 192.168.0.0/24, 10.10.10.1/32
latest handshake: <recent time>
transfer: <received> received, <sent> sent
persistent keepalive: every 25 seconds
```

A recent handshake proves that the peers can communicate and authenticate.

Traffic counters increasing in both directions show that encrypted traffic is being exchanged.

## 13. Testing in Layers

Testing was done incrementally.

First, test the ER605 WireGuard interface:

```bash
ping 10.10.10.1
```

This isolates the WireGuard path.

Then test a homelab node:

```bash
ping <homelab-node-ip>
```

This tests WireGuard plus ER605 forwarding/routing into the LAN.

Finally:

```bash
ssh <user>@<homelab-node-ip>
```

This verifies actual management access.

All homelab nodes became reachable directly from the Mac through WireGuard.

## 14. Reverse SSH Lesson

Before the VPN was working, reverse SSH demonstrated an important firewall concept.

A host inside a network can initiate an outbound SSH connection and create a path back through that connection.

That does not mean WireGuard prevents reverse SSH. WireGuard and egress filtering solve different problems.

### Security lesson

Blocking unsolicited inbound connections does not automatically prevent a compromised internal machine from establishing outbound tunnels.

Controlling outbound traffic is called **egress filtering** and can be important in higher-security environments.

## 15. Commands Worth Remembering

### Routing

```bash
route -n get <destination>
```

Shows which route, gateway, and interface macOS will use.

### WireGuard status

```bash
sudo wg show
```

Shows peers, endpoint, allowed IPs, handshake status, transfer counters, and keepalive configuration.

### Start WireGuard

```bash
sudo wg-quick up ~/lab-wg.conf
```

### Stop WireGuard

```bash
sudo wg-quick down ~/lab-wg.conf
```

### Test VPN endpoint

```bash
ping 10.10.10.1
```

### Test homelab reachability

```bash
ping <homelab-node-ip>
```

### SSH

```bash
ssh <user>@<homelab-node-ip>
```

## Main Takeaways

1. A router can participate in multiple IP networks at the same time.
2. Hosts use their routing tables to determine where packets should go.
3. A default gateway is a fallback next hop, not necessarily the packet's destination.
4. More-specific routes beat the default route.
5. WireGuard creates a virtual network interface and cryptographic peer relationship.
6. VPN addresses are additional virtual network identities.
7. Tunneling encapsulates actual traffic inside packets the underlying network can transport.
8. The outer packet gets encrypted traffic between tunnel endpoints.
9. The inner packet represents the traffic the application actually wants delivered.
10. The ER605 routes the decrypted inner packet using normal Layer 3 routing.
11. Troubleshooting is easier when each layer is tested independently.
