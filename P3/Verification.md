# On any router - confirm OSPF has built routes to the other loopbacks/links:
do sh ip route

# On the spine - confirm all 3 leafs connected (look for "Established" / uptime, not "Active"):
do sh bgp summary

# On any leaf - confirm its single session to the spine is up:
do sh bgp summary

# On any leaf or the spine - confirm EVPN routes exist (type 3 = "VTEP exists" announcements):
do sh bgp l2vpn evpn

# From host1 - confirms full VXLAN + BGP EVPN chain works end to end:
ping 20.1.1.2
ping 20.1.1.3

# On any leaf, after pinging - confirms automatic MAC discovery:
# new "route type 2" entries should now appear (one per host that sent traffic),
# alongside the type 3 entries seen earlier.
do sh bgp l2vpn evpn