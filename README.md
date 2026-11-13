# BGP, VXLAN, and EVPN Networking Lab

This repository contains a progressive network virtualization lab built with
[GNS3](https://www.gns3.com/), Docker, Linux networking tools, and
[FRRouting (FRR)](https://frrouting.org/). The lab starts with basic router and
host containers and evolves into a multi-router VXLAN fabric controlled by BGP
EVPN.

## Project goals

The main goal is to understand how modern data-center networks provide
Layer-2 connectivity across a routed Layer-3 infrastructure. In particular,
the project demonstrates how to:

- Build reproducible host and router environments with Docker.
- Configure Linux interfaces, bridges, loopbacks, and IP addressing.
- Establish an IP underlay between routers using OSPF.
- Create VXLAN tunnels to extend a host network across routed links.
- Compare manually configured unicast VXLAN with multicast-based discovery.
- Use BGP sessions to exchange reachability information between routers.
- Use EVPN to advertise MAC addresses and VXLAN tunnel endpoints instead of
  relying only on flood-and-learn behavior.
- Configure a spine as a BGP route reflector so additional leaf routers can
  join the fabric without configuring every leaf-to-leaf session.
- Verify connectivity and routing behavior with practical CLI commands.

## Lab progression

### Part 1: Docker and FRR foundations

Part 1 defines lightweight Docker environments for hosts and routers. The host
image is based on Alpine Linux, while the router image is based on FRR and
enables the routing daemons used by the lab.

See:

- [`P1/Readme.md`](P1/Readme.md) — networking and FRR terminology
- [`P1/Host1.md`](P1/Host1.md) — host image configuration
- [`P1/Router1.md`](P1/Router1.md) — router image configuration
- [`P1/cmd.md`](P1/cmd.md) — image build commands
- [`P1/P1.gns3project`](P1/P1.gns3project) — GNS3 project

### Part 2: VXLAN overlay networking

Part 2 connects two routers with an IP underlay and creates a VXLAN overlay
using VNI 10. The overlay is attached to a Linux bridge so hosts connected to
different routers can communicate as if they were on the same Layer-2 network.

The configuration demonstrates two VXLAN approaches:

1. **Static unicast VXLAN**, where each router is configured with the remote
   router's underlay address.
2. **Multicast VXLAN**, where routers join the same multicast group
   (`239.1.1.1`) and discover overlay peers dynamically.

See [`P2/README.md`](P2/README.md) and the router/host configuration files in
[`P2/`](P2/).

### Part 3: BGP EVPN VXLAN fabric

Part 3 expands the topology to one spine and three leaf routers:

```text
                 Spine / Route Reflector
                   1.1.1.1/32
                 /       |       \
          Leaf 1        Leaf 2        Leaf 3
        1.1.1.2/32    1.1.1.3/32    1.1.1.4/32
          Host 1        Host 2        Host 3
```

The routers use OSPF for underlay reachability and iBGP in AS 1 for EVPN
control-plane signaling. Each leaf acts as a VXLAN tunnel endpoint and
advertises VNI 10 through the `l2vpn evpn` address family. The spine uses a
BGP peer group, a listen range, and route-reflector-client configuration to
support dynamic leaf sessions.

See:

- [`P3/Readme.md`](P3/Readme.md) — VXLAN, BGP, and EVPN terminology
- [`P3/Router1.md`](P3/Router1.md) — spine configuration
- [`P3/Router2.md`](P3/Router2.md) — leaf 1 configuration
- [`P3/Router3.md`](P3/Router3.md) — leaf 2 configuration
- [`P3/Router4.md`](P3/Router4.md) — leaf 3 configuration
- [`P3/Host1.md`](P3/Host1.md), [`P3/Host2.md`](P3/Host2.md), [`P3/Host3.md`](P3/Host3.md)
- [`P3/Verification.md`](P3/Verification.md) — validation commands
- [`P3/P3.gns3project`](P3/P3.gns3project) — GNS3 project

## Requirements

- GNS3
- Docker
- Linux networking tools (`ip`, `bridge`, and `ping`)
- FRRouting through the router image used in Part 1

The Docker daemon must be running before starting the GNS3 projects.

## Getting started

1. Clone the repository.
2. Build the host and router images using the commands in
   [`P1/cmd.md`](P1/cmd.md).
3. Open the desired `.gns3project` file in GNS3.
4. Start with Part 1, then continue through Part 2 and Part 3.
5. Apply the commands in each host and router document to the corresponding
   node.
6. Verify each layer before moving upward:
   - Underlay IP reachability
   - OSPF routes to loopbacks
   - BGP session state
   - EVPN routes
   - End-to-end host connectivity

## Verification

Useful checks for the Part 3 fabric include:

```text
do sh ip route
do sh bgp summary
do sh bgp l2vpn evpn
```

After configuring the hosts with addresses in `20.1.1.0/24`, test the overlay
from Host 1:

```text
ping 20.1.1.2
ping 20.1.1.3
```

Successful pings confirm the complete path: host bridge, VXLAN interface,
underlay routing, BGP EVPN signaling, and the remote host bridge. After traffic
has been exchanged, EVPN type 2 routes should appear for learned host MAC
addresses, alongside type 3 routes that identify reachable VTEPs.

## Key concepts

- **Underlay:** The routed IP network used to transport VXLAN packets.
- **Overlay:** The virtual Layer-2 network presented to the hosts.
- **VXLAN:** Encapsulates Ethernet frames in UDP/IP packets.
- **VNI:** Identifies a VXLAN segment; this lab uses VNI 10.
- **BGP EVPN:** Distributes MAC and VTEP reachability information.
- **Spine:** Provides underlay connectivity and reflects EVPN routes.
- **Leaf/VTEP:** Connects hosts to the overlay and encapsulates VXLAN traffic.

## Expected outcome

At the end of the project, hosts attached to different leaf routers should be
able to communicate over the same logical Layer-2 subnet. The control plane
should provide scalable VTEP and MAC discovery through BGP EVPN, while OSPF
provides the IP reachability required by the VXLAN data plane.
