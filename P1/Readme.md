## General / cross-project terms

- **GNS3** — a network simulator that runs real router/host operating systems (as
  Docker containers or VMs) and connects them with virtual cables, so you can
  build and test a network without physical hardware.
- **Layer 2 (L2)** — the Ethernet/MAC-address layer. Delivery within one local
  network ("room") — a switch forwards frames based on MAC addresses, no routing
  decisions needed.
- **Layer 3 (L3)** — the IP-address layer. Delivery across different networks —
  requires a router to decide the best path based on IP addresses.
- **Router** — a device that moves traffic between different Layer 3 networks
  (subnets), making path decisions.
- **Switch** — a device that connects machines within one Layer 2 network; anything
  plugged into it can reach each other directly via MAC addresses, no routing needed.
- **Interface** — a network connection point on a device (physical, like `eth0`, or
  virtual, like a loopback or a VXLAN interface).
- **Loopback interface (`lo`)** — a virtual interface that always stays up, since it
  isn't tied to any physical cable. Used as a router's stable identity (e.g. for
  OSPF router ID or BGP sessions), because it survives even if one physical link fails.
- **Subnet mask / CIDR (`/24`, `/30`, `/32`)** — defines how many addresses count as
  "the same local group." `/24` = a normal LAN-sized block; `/30` = exactly 2 usable
  addresses (a point-to-point link); `/32` = exactly one single address (used for
  loopbacks).

---

## Part 1 — GNS3 configuration with Docker

- **Docker image** — a packaged blueprint (filesystem + default startup command)
  used to create containers; built from a `Dockerfile`.
- **Docker container** — a running instance of an image; behaves like an isolated
  mini Linux machine.
- **BusyBox** — a single small program bundling simplified versions of many common
  command-line tools (`ls`, `ping`, `ip`, etc.), used in minimal environments.
  Alpine Linux ships with BusyBox-based utilities by default.
- **Packet routing software** — software that decides which path traffic takes and
  maintains the routing table; `zebra` (part of FRR) plays this role, feeding
  routes into the kernel while other daemons (bgpd, ospfd) supply it information.
- **FRR (FRRouting)** — an open-source suite of routing daemons (zebra, bgpd,
  ospfd, isisd, etc.) that together turn a Linux machine into a functional router.
- **bgpd** — the FRR daemon implementing BGP; handles BGP sessions and route/EVPN
  exchange.
- **ospfd** — the FRR daemon implementing OSPF; handles neighbor discovery and
  shortest-path route computation.
- **isisd** — the FRR daemon implementing IS-IS, a link-state routing protocol
  similar to OSPF, more common in large carrier/ISP backbones.
- **`vtysh`** — FRR's interactive configuration shell, used to configure all FRR
  daemons through one unified command-line interface.

---