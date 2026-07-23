FROM frrouting/frr:latest
# Base image: official FRRouting image - already contains zebra, bgpd,
# ospfd, isisd and the other FRR daemons required by the subject.
# zebra itself is included and runs by default - it's the core daemon
# that programs routes into the kernel; bgpd/ospfd/isisd all depend on it.

RUN apk add --no-cache iproute2 \
tcpdump \
iputils \
bash
# Same networking/debugging tools as the host image, added on top of FRR:
# iproute2 for interface/VXLAN/bridge config, tcpdump for packet capture,
# iputils for ping, bash for a proper shell.

RUN echo 'service integrated-vtysh-config' > /etc/frr/vtysh.conf
# Tell FRR to keep all daemon configs merged into one single file
# (/etc/frr/frr.conf) instead of separate per-daemon config files -
# this is why "do wr mem" earlier saved everything to frr.conf directly.

RUN sed -i "s/bgpd=no/bgpd=yes/" /etc/frr/daemons && \
    sed -i "s/ospfd=no/ospfd=yes/" /etc/frr/daemons && \
    sed -i "s/isisd=no/isisd=yes/" /etc/frr/daemons
# FRR ships with every daemon disabled by default in /etc/frr/daemons.
# Enable exactly the ones the subject requires:
# bgpd  -> BGP routing (needed for Part 3's EVPN work)
# ospfd -> OSPF routing (needed for underlay reachability, Parts 2 & 3)
# isisd -> IS-IS routing engine (required by the subject, even if unused later)