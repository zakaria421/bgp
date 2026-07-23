
# =============================================================
# SECTION 0 - Underlay configuration
# =============================================================
# Assign the underlay IP to eth0. This is the "real" address used
# to actually deliver VXLAN-encapsulated packets to routeur2.
# This has nothing to do with the hosts' network - it only needs
# to let the two routers reach each other over plain IP.
ip addr add 10.1.1.1/24 dev eth0
ip link set eth0 up

# Quick sanity check before touching VXLAN at all:
   ping 10.1.1.2   #-> must work, otherwise nothing built on top
#                      of it (VXLAN, bridge) can possibly work.


# =============================================================
# SECTION 1 - VXLAN interface: STATIC (unicast) mode
# =============================================================
# Create the VXLAN interface:
#   id 10            -> VNI (VXLAN Network Identifier). Acts like a
#                        label on the outer envelope so the far end
#                        knows which overlay network a frame belongs
#                        to (useful when many overlays share one link).
#   remote 10.1.1.2  -> hardcodes routeur2's underlay address as
#                        the ONE destination to send encapsulated
#                        frames to. This is what makes this "static":
#                        the peer is fixed by hand.
#   dstport 4789     -> standard UDP port used for VXLAN traffic
#                        (IANA-assigned, per RFC 7348).
#   dev eth0         -> encapsulated packets are sent out via eth0,
#                        the interface facing the underlay network.
ip link add name vxlan10 type vxlan id 10 remote 10.1.1.2 dstport 4789 dev eth0

# Turn the tunnel interface on.
ip link set vxlan10 up

# Give the tunnel interface its own IP purely so we can test the raw
# tunnel directly (ping router-to-router through the tunnel) before
# adding bridge complexity on top. Hosts never use this address.
ip addr add 30.1.2.3/24 dev vxlan10

# Verification:
  ip link show vxlan10   #-> should show the interface UP with the
#                              VNI and remote peer listed
   ping 30.1.2.4           #-> tests the tunnel itself (router to
#                              router), before hosts are involved


# =============================================================
# SECTION 2 - Bridge: connects the host-facing link to the tunnel
# =============================================================
# eth1 (facing the host) and vxlan10 (facing the tunnel) know
# nothing about each other yet. The bridge acts as a virtual
# switch: anything arriving on one port is forwarded out the
# other port.
ip link add name br0 type bridge
ip link set dev br0 up

# Plug the VXLAN tunnel in as one port on the virtual switch.
ip link set vxlan10 master br0

# Plug the host-facing interface in as the other port.
ip link set eth1 master br0
ip link set eth1 up

# Verification:
   bridge link show     #-> should list both eth1 and vxlan10 as
#                            ports attached to br0
   #(from host1) 
   ping 30.1.1.2   #-> full end-to-end test


# =============================================================
# SECTION 3 - Switch to DYNAMIC (multicast) mode
# =============================================================
# The "remote" setting can't be changed on an existing interface,
# so we delete and recreate vxlan10 (and br0, since it loses its
# vxlan10 port once vxlan10 is deleted).
ip link delete br0
ip link delete vxlan10

# Recreate the VXLAN interface using a multicast GROUP instead of
# a single hardcoded "remote" peer:
#   group 239.1.1.1 -> any router that joins this same multicast
#                       group for VNI 10 is automatically able to
#                       send/receive with this router - no manual
#                       peer list to maintain. Adding a 3rd, 4th...
#                       router later just means it also joins this
#                       group; nothing here needs to change.
#   ttl auto        -> lets the kernel pick a sensible Time-To-Live
#                       for the outer multicast packets automatically.
ip link add name vxlan10 type vxlan id 10 dstport 4789 group 239.1.1.1 dev eth0 ttl auto
ip link set vxlan10 up
ip addr add 30.1.2.3/24 dev vxlan10

# Rebuild the bridge exactly as before.
ip link add name br0 type bridge
ip link set dev br0 up
ip link set vxlan10 master br0
ip link set eth1 master br0
ip link set eth1 up

# Verification:
   ip -d link show vxlan10   #-> should now show "group 239.1.1.1"
#                                 instead of a "remote" peer
   bridge fdb show            #-> MAC address table: shows MACs
#                                  learned on eth1 (local host) vs
#                                  vxlan10 (far router, via tunnel)
#   (from host1) 
    ping 30.1.1.2   #-> full end-to-end test again
