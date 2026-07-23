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

hostname zbentale-3
no ipv6 forwarding

interface eth0
 ip address 10.1.1.6/30
 # Underlay IP, link toward the spine (different /30 subnet - separate physical link).
 ip ospf area 0
exit

interface lo
 ip address 1.1.1.3/32
 # This leaf's own stable loopback identity.
 ip ospf area 0
exit

router bgp 1
 neighbor 1.1.1.1 remote-as 1
 neighbor 1.1.1.1 update-source lo
 address-family l2vpn evpn
  neighbor 1.1.1.1 activate
  advertise-all-vni
 exit-address-family
exit

router ospf
exit