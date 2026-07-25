- **VXLAN (Virtual Extensible LAN, RFC 7348)** — a tunneling technology that wraps
  Ethernet frames inside UDP/IP packets, letting two physically separated machines
  behave as if they're on the same local Layer 2 network.
- **Overlay network** — the "fake" shared network the tunnel creates (what the
  hosts believe they're on).
- **Underlay network** — the real, physically routed network the tunnel actually
  travels across (invisible to the hosts).
- **VNI (VXLAN Network Identifier)** — the numeric label (e.g. `id 10`) identifying
  which overlay network a VXLAN packet belongs to; same role as a VLAN tag, but
  with a much larger range (~16 million vs ~4094 for VLAN).
- **VLAN** — an older technology that tags frames to logically separate traffic on
  the *same physical* local network; unlike VXLAN, it doesn't extend across a
  routed network on its own, and supports far fewer IDs (max 4094).
- **Encapsulation** — wrapping one packet/frame inside another (the "envelope
  inside an envelope" idea); this is exactly what a VXLAN interface does before
  sending traffic across the underlay.
- **UDP port 4789** — the standard port VXLAN traffic uses to travel across the
  underlay network (IANA-assigned).
- **Bridge** — a virtual switch built in software; connects two interfaces (e.g. a
  host-facing interface and a VXLAN tunnel interface) so traffic can flow between
  them.
- **Static / unicast VXLAN mode** — the tunnel destination is hardcoded by hand
  using `remote <ip>`; simple, but doesn't scale to many routers.
- **Dynamic / multicast VXLAN mode** — the tunnel uses a multicast `group` address
  instead of a fixed peer; routers auto-discover each other by joining the same
  group, so adding new routers doesn't require editing existing ones.
- **Multicast** — delivery to only the members of a specific "group" (like a
  mailing list), as opposed to unicast (one specific destination) or broadcast
  (everyone).
- **Broadcast** — delivery to everyone within a local network, with no filtering.
- **Multicast group address (e.g. `239.1.1.1`)** — an IP address in the range
  reserved for private/local multicast groups.
- **TTL (Time To Live)** — a counter on IP packets that decreases at each router
  hop and causes the packet to be dropped at 0; prevents infinite loops. `ttl auto`
  lets the kernel pick a sensible default for multicast VXLAN traffic.
- **MAC address table / FDB (Forwarding Database)** — the table a bridge/switch
  keeps, mapping MAC addresses to the port they were learned on; viewed with
  `bridge fdb show`.
- **Flood and learn** — the default behavior of a switch/bridge: when it doesn't
  know where a MAC address is, it floods the frame to all ports, then learns the
  real location from the response.