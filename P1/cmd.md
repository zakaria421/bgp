# Building the Docker images
docker build -t host_dockerfile -f host_dockerfile .
docker build -t router_dockerfile -f router_dockerfile .