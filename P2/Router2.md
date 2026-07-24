
# ============================================================
# SECTION 0 - Underlay configuration
# =============================================================
ip addr add 10.1.1.2/24 dev eth0
ip link set eth0 up

# Sanity check:
   ping 10.1.1.1   #-> must work before continuing


# =============================================================
# SECTION 1 - VXLAN interface: STATIC (unicast) mode
# =============================================================
# Same as routeur-1, but "remote" points back at
# routeur-1's underlay address (10.1.1.1).
ip link add name vxlan10 type vxlan id 10 remote 10.1.1.1 dstport 4789 dev eth0
ip link set vxlan10 up
ip addr add 30.1.2.4/24 dev vxlan10

# Verification:
   ip link show vxlan10
    ping 30.1.2.3   #-> tests the tunnel itself, router to router


# =============================================================
# SECTION 2 - Bridge: connects the host-facing link to the tunnel
# =============================================================
ip link add name br0 type bridge
ip link set dev br0 up
ip link set vxlan10 master br0
ip link set eth1 master br0
ip link set eth1 up

# Verification:
   bridge link show
#   (from host-2) 
    ping 30.1.1.1   #-> full end-to-end test


# =============================================================
# SECTION 3 - Switch to DYNAMIC (multicast) mode
# =============================================================
ip link delete br0
ip link delete vxlan10

# Same multicast group as routeur-1 - this is what lets the
# two (or more) routers auto-discover each other for VNI 10.
ip link add name vxlan10 type vxlan id 10 dstport 4789 group 239.1.1.1 dev eth0 ttl auto
ip link set vxlan10 up
ip addr add 30.1.2.4/24 dev vxlan10

ip link add name br0 type bridge
ip link set dev br0 up
ip link set vxlan10 master br0
ip link set eth1 master br0
ip link set eth1 up

# Verification:
   ip -d link show vxlan10   #-> should show "group 239.1.1.1"
   bridge fdb show
#   (from host-2) 
    ping 30.1.1.1   #-> full end-to-end test again

#if brodcast is not showing, check the following:
   ip neigh flush all #in host-1, host-2
