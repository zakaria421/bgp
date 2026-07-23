vtysh
conf t

hostname zbentale-1
no ipv6 forwarding

interface eth0
 ip address 10.1.1.1/30
 # Underlay link toward Routeur 2.
exit

interface eth1
 ip address 10.1.1.5/30
 # Underlay link toward Routeur 3.
exit

interface eth2
 ip address 10.1.1.9/30
 # Underlay link toward Routeur 4.
exit

interface lo
 ip address 1.1.1.1/32
 # Spine's own loopback - this is what every leaf points its "neighbor" at.
exit

router bgp 1
 neighbor ibgp peer-group
 # Define a reusable template ("ibgp") for settings shared by all leaf sessions.
 neighbor ibgp remote-as 1
 # Any peer using this template is expected to be in AS 1.
 neighbor ibgp update-source lo
 # Use the spine's own loopback as the source for these sessions.
 bgp listen range 1.1.1.0/29 peer-group ibgp
 # Passively accept BGP connections from any IP in 1.1.1.0/29, auto-applying
 # the ibgp template - this is what makes leaf discovery "dynamic": add a new
 # leaf later and nothing here needs to change.

 address-family l2vpn evpn
  neighbor ibgp activate
  # Turn on EVPN exchange for all sessions using the ibgp template.
  neighbor ibgp route-reflector-client
  # Allow the spine to relay (reflect) EVPN routes it learns from one leaf
  # to the other leafs - without this, announcements would stop at the spine.
 exit-address-family
exit

router ospf
 network 0.0.0.0/0 area 0
 # Enable OSPF on every configured interface (all 3 links + loopback) at once.
exit