FROM alpine:latest
# Base image: Alpine Linux, small and lightweight, satisfies the
# "busybox or equivalent" requirement for the host image.

RUN apk add --no-cache iproute2 \
tcpdump \
iputils \
bash
# Install networking tools needed for the lab:
# iproute2  -> gives us "ip addr", "ip link" etc. (core commands used throughout)
# tcpdump   -> lets us capture/inspect traffic (used for verification screenshots)
# iputils   -> provides "ping" for connectivity testing
# bash      -> proper shell (we found the aux/default console misbehaved earlier)
# --no-cache avoids leaving the package index cached in the image, keeping it smaller.

CMD ["/bin/bash"]
# Default command when the container starts: drop into a bash shell.