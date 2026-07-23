ip link add name vxlan10 type vxlan id 10 dstport 4789
# Create VXLAN interface, VNI 10, standard VXLAN UDP port.
# No "remote" or "group" here - BGP EVPN will supply peer info dynamically.

ip link set vxlan10 up
# Turn the VXLAN interface on.

ip link add name br0 type bridge
# Create a virtual switch to connect the host-facing link and the VXLAN tunnel.

ip link set dev br0 up
# Turn the bridge on.

ip link set vxlan10 master br0
# Attach the VXLAN tunnel as a port on the bridge.

ip link set eth1 master br0
# Attach the host-facing interface as the other port on the bridge.

# --- Verification ---
ip link show vxlan10        # confirm vxlan10 is UP
bridge link show            # confirm both eth1 and vxlan10 are bridge ports

vtysh
conf t

hostname zbentale-2
# Set this router's name 

no ipv6 forwarding
# Disable IPv6 routing - this lab only uses IPv4.

interface eth0
 ip address 10.1.1.2/30
 # Underlay IP, link toward the spine.
 ip ospf area 0
 # Enable OSPF on this link so this router and the spine can find each other.
exit

interface lo
 ip address 1.1.1.2/32
 # Stable loopback identity - always reachable, used for the BGP session.
 ip ospf area 0
 # Advertise the loopback via OSPF so other routers can route to it.
exit

router bgp 1
 # This router belongs to AS 1.
 neighbor 1.1.1.1 remote-as 1
 # Establish a BGP session with the spine (also AS 1).
 neighbor 1.1.1.1 update-source lo
 # Use this router's own loopback as the source for that session (stable, link-independent).

 address-family l2vpn evpn
  # Use this BGP session to also exchange EVPN (MAC-location) info.
  neighbor 1.1.1.1 activate
  # Turn on the EVPN exchange specifically toward the spine.
  advertise-all-vni
  # Automatically announce all locally known VNIs and their MAC addresses.
 exit-address-family
exit

router ospf
exit
# (No extra network statements needed - "ip ospf area 0" on each interface already covers it.)