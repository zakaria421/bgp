- **BGP (Border Gateway Protocol)** — a protocol two routers use to exchange
  routing/reachability information over a session; the protocol that connects
  different networks (Autonomous Systems) across the internet.
- **AS (Autonomous System)** — an independently administered network, identified
  by an AS number; in this lab, all routers share one AS (`1`), since they belong
  to the same "organization."
- **BGP session / neighbor relationship** — a direct connection between two
  specific routers configured to exchange BGP information (`neighbor <ip>
  remote-as <asn>`).
- **`update-source lo`** — tells a router to use its own loopback address as the
  source for a BGP session, keeping the session stable even if one physical link
  fails.
- **EVPN (Ethernet VPN, RFC 7432)** — a BGP address-family dedicated to
  proactively announcing MAC address locations and VXLAN endpoint (VTEP)
  information, replacing "flood and learn" with direct announcements.
- **`address-family l2vpn evpn`** — the BGP configuration block that activates
  EVPN-specific exchange on a session.
- **`advertise-all-vni`** — tells a router to automatically announce all locally
  known VNIs and the MAC addresses it discovers on them, over EVPN.
- **VTEP (VXLAN Tunnel Endpoint)** — the router-side "end" of a VXLAN tunnel; the
  device that actually encapsulates/decapsulates frames. In this lab, each leaf
  router is a VTEP, identified by its loopback address.
- **Route reflector (RR)** — a router explicitly allowed to relay (reflect) BGP
  routes it learns from one client to its other clients, overriding BGP's default
  no-relay loop-prevention rule. The spine plays this role in Part 3.
- **`route-reflector-client`** — the setting that marks specific neighbors as
  clients a route reflector is allowed to relay information between.
- **Peer-group** — a named template (e.g. `ibgp`) that bundles shared BGP neighbor
  settings, so they don't need to be repeated for every individual neighbor.
- **`bgp listen range`** — tells a router to passively accept BGP connections from
  any address within a given range, automatically applying a peer-group template;
  this is what makes leaf discovery "dynamic" (new leafs don't require spine
  reconfiguration).
- **Dynamic neighbor** — a BGP neighbor that connected via a listen range rather
  than being explicitly configured by IP; shown with a `*` in `show bgp summary`.
- **EVPN route type 3** — an automatic "I exist" announcement made by each VTEP as
  soon as a VNI is configured; contains no MAC info, just VTEP reachability.
- **EVPN route type 2** — an announcement created once a real device (host) sends
  traffic; says "this specific MAC address is reachable behind this VTEP." This is
  the automatic MAC-learning behavior EVPN provides.
- **`show bgp summary`** — displays the state of BGP sessions (established or not,
  how many routes received/sent).
- **`show bgp l2vpn evpn`** — displays the actual EVPN routes (type 2 and type 3)
  learned by a router.