
# Assign the overlay IP address to eth1 (the interface facing
# routeur-2).
ip addr add 30.1.1.2/24 dev eth1

# Make sure the interface is up.
ip link set eth1 up

# -------------------------------------------------------------
# Verification commands (not part of the config, just for testing)
# -------------------------------------------------------------
ip addr show eth1       # -> confirm 30.1.1.2/24 is present and UP
ping 30.1.1.1             # -> should reach host-1
