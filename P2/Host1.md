

# Assign the overlay IP address to eth1 (the interface facing
# routeur-1). This is the address host-1 uses to reach
# host-2 directly, as if both were plugged into the same
# switch, even though a routed network + VXLAN tunnel actually
# separates them.
ip addr add 30.1.1.1/24 dev eth1

# Make sure the interface is up.
ip link set eth1 up

# -------------------------------------------------------------
# Verification commands (not part of the config, just for testing)
# -------------------------------------------------------------
 ip addr show eth1        # -> confirm 30.1.1.1/24 is present and UP
 ping 30.1.1.2             # -> should reach host-2 once the
#                              VXLAN tunnel + bridge are configured
#                              on both routers
