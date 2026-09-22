# Docker Networking

## Interview Summary

Docker networking connects containers to each other, to the Docker host, and, when required, to external networks. A container normally has its own network namespace, interfaces, IP address, routing table, and DNS configuration. Docker creates the required host-side networking and connects the container to a network driver.

A good interview answer is:

> Containers are isolated by default, but they can communicate when attached to the same Docker network. A user-defined bridge network is the usual choice for a single Docker host because it provides container-to-container connectivity, automatic DNS-based service discovery, network-level separation, and controlled host exposure through published ports.

Networking does not provide complete security by itself. Use least-privilege network membership, firewalls, TLS, authentication, authorization, image hardening, and a production network policy.

## The Linux Networking Path

For a normal bridge network, the traffic path is approximately:

```text
Container process
    |
    | eth0 inside the container network namespace
    |
Container-side veth <==== virtual Ethernet pair ====> vethXXXX on host
                                                       |
                                                       | attached to bridge
                                                       v
                                             docker0 or a user-defined bridge
                                                       |
                                  iptables/nftables, routing, and optional NAT
                                                       |
                                     Host interface (eth0/ens5) or another network
```

Important terms:

- **Network namespace:** isolates interfaces, routes, ports, and network state for a process group.
- **veth pair:** two connected virtual Ethernet interfaces. One endpoint is placed in the container namespace as `eth0`; the peer remains on the host.
- **Linux bridge:** a virtual Layer 2 switch. It forwards Ethernet frames between attached veth interfaces and can connect to host networking rules.
- **Docker bridge:** the Docker-managed Linux bridge used by the `bridge` driver. The default network commonly uses `docker0`; a user-defined bridge normally gets a separate bridge device.
- **NAT/masquerading:** allows private container addresses to reach external networks through the host address. Published ports add forwarding from a host port to a container port.

The container does not directly use the host's physical interface. Docker creates the veth pair and attaches the host endpoint to a bridge. If the veth, bridge, routing, or firewall rules are manually deleted or damaged, the container can lose connectivity until Docker recreates or repairs the endpoint. Do not modify Docker-managed interfaces manually on a production host.

## Container-to-Container and Host-to-Container Traffic

### Container to container

Two containers attached to the same user-defined bridge can communicate using the other container's name and exposed service port:

```bash
curl http://finance:80
```

The `EXPOSE` instruction is metadata; it does not publish a port or make a service reachable from outside the host. The application must also listen on the correct interface, usually `0.0.0.0`, rather than only `127.0.0.1` inside the container.

### Host to container

The host reaches a container through its container IP or, preferably, a published host port:

```bash
docker run -d --name web -p 8080:80 nginx:latest
curl http://localhost:8080
```

`-p 8080:80` means host port `8080` forwards to container port `80`. Without `-p`, a bridge-network container is normally not reachable from clients outside the Docker host, even though containers on the same Docker network can reach it.

Bind the published port to a specific host address when appropriate:

```bash
# Only listen on localhost
docker run -d --name local-web -p 127.0.0.1:8080:80 nginx:latest

# Listen on all host interfaces; protect the host with a firewall/security group
docker run -d --name public-web -p 0.0.0.0:8080:80 nginx:latest
```

## Docker Network Drivers

### Bridge

The default choice for containers running on one Docker host. Each container gets a private address on the bridge network. User-defined bridges provide embedded DNS, scoped network membership, aliases, and better isolation than the legacy default bridge.

Advantages: isolation from the host network, simple service discovery, port publishing, and good support for multi-container applications.

Limitations: the network is local to one Docker host, and it is not a substitute for application authentication or a production network policy.

### Host

The container shares the host's network namespace. There is no separate container IP and no Docker bridge or veth path for that container. The application binds directly to host interfaces and ports.

```bash
docker run -d --name host-web --network host nginx:latest
```

Advantages: lower network overhead and direct access to host networking.

Disadvantages: poor network isolation, host-port conflicts, platform limitations, and greater exposure if the application is insecure. `-p` is ignored or unnecessary in host mode because the container uses host ports directly.

### None

Disables normal networking and gives the container only loopback unless additional interfaces are configured manually. Useful for highly isolated jobs or tests that must not communicate over a network.

```bash
docker run --rm -it --network none alpine:latest sh
```

### Overlay

Connects containers across multiple Docker hosts. It is commonly used with Docker Swarm or an orchestrator. It requires cross-host control/data-plane configuration and is unnecessary for a single-host application.

```bash
# Requires Swarm mode and is normally created as an attachable network
docker network create --driver overlay --attachable app-overlay
```

### Macvlan and ipvlan

These drivers connect containers more directly to the physical network. Macvlan gives containers distinct MAC addresses; ipvlan can reduce MAC-address usage and offers different Layer 2 or Layer 3 behavior.

Advantages: useful for legacy applications that need to appear as first-class network devices or require direct Layer 2/3 integration.

Disadvantages: more complex routing and switch configuration, security concerns, host-to-container communication caveats, and reduced portability. They are not the default choice for ordinary web applications.

### Driver comparison

| Driver | Main use | Isolation | Scope |
| --- | --- | --- | --- |
| `bridge` | Containers on one host | Good by default; stronger with user-defined networks | Local |
| `host` | Performance or host-network software | Minimal | Local |
| `none` | Network-isolated jobs | Highest normal isolation | Local |
| `overlay` | Multi-host container networks | Depends on configuration and orchestration | Swarm/multi-host |
| `macvlan` | Direct Layer 2 identity | Different from bridge; requires network planning | Local |
| `ipvlan` | Direct Layer 2/3 integration with fewer MACs | Different from bridge; requires network planning | Local |

For most applications on one Docker host, a **user-defined bridge** is the best starting point. For multi-host production deployments, use the networking model provided by the chosen orchestrator or cloud platform.

## Default Bridge versus User-Defined Bridge

The default `bridge` network is created by Docker automatically. Containers attached to it can communicate by IP, but legacy behavior does not provide the same convenient name-based DNS service as a user-defined bridge. The old `--link` mechanism is deprecated; use a user-defined network instead.

A user-defined bridge is better because it provides:

- Automatic DNS resolution by container name and network alias.
- Explicit membership: only attached containers participate.
- Better separation between application stacks.
- Ability to connect or disconnect containers at runtime.
- Custom subnet, gateway, IP range, and driver options.

The fact that two containers share a bridge does not automatically mean their data is exposed to the internet. External reachability requires a published port, host routing, or another path. However, any compromised container on the same network may attempt to connect to services that are listening there, so use separate networks and application-level controls.

## CIDR and Docker IP Address Management

**CIDR** means **Classless Inter-Domain Routing**. It represents an IP network as an address followed by a prefix length:

```text
172.18.0.0/16
```

The `/16` says that the first 16 bits identify the network and the remaining 16 bits identify host addresses. In IPv4, the subnet mask is `255.255.0.0`. The network contains $2^{32-16}=65,536$ total addresses, although Docker reserves or uses some addresses for network functions and not every address is assignable in every configuration.

Common examples:

| CIDR | Netmask | Total IPv4 addresses | Typical use |
| --- | --- | ---: | --- |
| `172.18.0.0/16` | `255.255.0.0` | 65,536 | Large Docker bridge range |
| `172.18.0.0/24` | `255.255.255.0` | 256 | Small application network |
| `10.0.0.0/8` | `255.0.0.0` | 16,777,216 | Private enterprise range |
| `192.168.1.0/24` | `255.255.255.0` | 256 | Common private LAN |

Docker's IPAM (IP Address Management) allocates addresses from the configured subnet and assigns a gateway to the bridge. Avoid overlapping the Docker subnet with the host LAN, VPN, cloud VPC, or another route. Overlap creates ambiguous routing and hard-to-debug connectivity failures.

Create a custom subnet and gateway:

```bash
docker network create \
  --driver bridge \
  --subnet 172.30.0.0/24 \
  --gateway 172.30.0.1 \
  app-network
```

Inspect its IPAM configuration:

```bash
docker network inspect app-network
```

Use a static IP only when there is a strong operational reason. Prefer service names because containers are replaceable and IP addresses can change.

```bash
docker run -d --name finance \
  --network app-network \
  --ip 172.30.0.10 \
  nginx:latest
```

## Docker Network Commands

```bash
# List networks
docker network ls

# Create a user-defined bridge network
docker network create app-network

# Create with a name, driver, subnet, and gateway
docker network create \
  --driver bridge \
  --subnet 172.30.0.0/24 \
  --gateway 172.30.0.1 \
  --label environment=dev \
  secure-network

# Inspect driver, IPAM, containers, and options
docker network inspect secure-network

# Run a container directly on a network
docker run -d --name finance --network secure-network nginx:latest

# Connect an existing container to another network
docker network connect secure-network login-container

# Add an alias on that network
docker network connect --alias payments secure-network login-container

# Connect with a static IP on a custom subnet
docker network connect --ip 172.30.0.20 app-network login-container

# Disconnect a container from a network
docker network disconnect secure-network login-container

# Remove a network; it must not have connected containers
docker network rm secure-network

# Remove all unused custom networks
docker network prune

# Show Docker's network-related daemon information
docker info
```

Helpful container inspection commands:

```bash
# Show network mode, IP addresses, gateways, aliases, and ports
docker inspect --format '{{json .NetworkSettings.Networks}}' finance

# Show a compact IP address
docker inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' finance

# Display listening ports and published mappings
docker port finance

# Show all containers attached to a network
docker network inspect secure-network --format '{{json .Containers}}'
```

`docker network rm` fails when containers are still attached. Disconnect or remove the containers first, then remove the network. Never remove a network merely to fix an application problem without checking its attached services.

## Three-Container Custom Bridge Lab

This lab creates a simple three-tier layout:

- `login`: client-facing web container.
- `finance`: internal application service.
- `database`: internal data service.

The `frontend` network contains `login` and `finance`. The `backend` network contains `finance` and `database`. Because `login` is not attached to `backend`, it has no network path to `database`. This is network segmentation, not a replacement for database authentication.

```bash
# Clean up lab resources if they already exist
docker rm -f login finance database 2>/dev/null || true
docker network rm frontend backend 2>/dev/null || true

# Create two isolated user-defined bridge networks
docker network create --driver bridge frontend
docker network create --driver bridge backend

# Start an internal database placeholder
docker run -d --name database \
  --network backend \
  busybox:latest sh -c 'while true; do echo database-ready; sleep 30; done'

# Start the finance service on the backend network
docker run -d --name finance \
  --network backend \
  nginx:latest

# Attach finance to the frontend network as well
docker network connect frontend finance

# Start the login service on the frontend network and publish it to the host
docker run -d --name login \
  --network frontend \
  -p 8080:80 \
  nginx:latest

# Test host-to-login connectivity
curl http://localhost:8080

# Test name-based resolution from login to finance
docker run --rm --network frontend busybox:latest ping -c 2 finance

# Inspect both network memberships
docker network inspect frontend
docker network inspect backend

# Remove the lab
docker rm -f login finance database
docker network rm frontend backend
```

The `busybox ping` check verifies network reachability, not that an HTTP service is healthy. For HTTP testing, use a temporary image that contains `curl`:

```bash
docker run --rm --network frontend curlimages/curl:latest http://finance
```

Do not install troubleshooting tools into a running production container as a first response. Use a temporary diagnostic container or an image designed for debugging, and keep production images minimal.

## Reproducing the Original Experiment

The original command had a typo (`dokcer`) and correctly showed the difference between a container on the default bridge and one on a custom network:

```bash
docker run -d --name login-container nginx:latest
docker exec -it login-container /bin/bash

docker network create secure-network
docker run -d --name finance --network secure-network nginx:latest
docker network ls
docker inspect finance
```

If `finance` receives an address such as `172.18.0.2`, that address belongs to the custom bridge subnet shown in `docker inspect`. A container on another network is not automatically a member of that network. Testing the IP directly may fail because of network isolation, firewall rules, or the service's listening configuration. Test the intended name and port from a container attached to the same network:

```bash
docker run --rm --network secure-network curlimages/curl:latest http://finance
```

To allow an existing container to communicate with `finance`, attach it to `secure-network`:

```bash
docker network connect secure-network login-container
docker exec login-container getent hosts finance
```

The address `172.18.0.2` is an implementation detail. Do not hard-code it in application configuration; use the service name `finance`.

## DNS and Service Discovery

Docker's embedded DNS resolver makes container names and aliases resolvable on user-defined networks. Applications should use a stable service name:

```text
http://finance:80
```

When a container is recreated, its IP may change while its network name remains stable. This is why name-based discovery is more reliable than manually recording container IP addresses.

DNS troubleshooting:

```bash
docker exec login getent hosts finance
docker exec login cat /etc/resolv.conf
docker inspect --format '{{json .NetworkSettings.Networks}}' login
```

If DNS works but the connection fails, check the destination port, process listening address, container health, network membership, and firewall. If DNS fails, check that both containers share a user-defined network and that the name or alias is correct.

## Security and Isolation

Use a layered approach:

- Put only services that need to communicate on the same user-defined network.
- Keep databases on an internal network without published ports.
- Publish only the reverse proxy or required ingress service.
- Use `--internal` for a network that should not have external connectivity through Docker's normal routing path.
- Restrict host firewall and cloud security-group rules.
- Use TLS, authentication, and authorization between services.
- Avoid `--network host` unless the application specifically requires host networking.
- Avoid `--privileged`; network administration capabilities can be granted narrowly when truly required.

Create an internal network:

```bash
docker network create --driver bridge --internal backend-internal
```

`--internal` helps prevent external connectivity through that network, but it is not a complete security boundary. Validate the behavior against your Docker version, host firewall, and application requirements.

## Troubleshooting Workflow

When a container cannot communicate, work from the closest layer outward:

```bash
# Is the container running?
docker ps -a

# Is the process listening, and what did it log?
docker logs <container>
docker exec <container> ss -lnt

# Is the container attached to the expected network?
docker inspect --format '{{json .NetworkSettings.Networks}}' <container>

# Can DNS resolve the target name?
docker exec <source> getent hosts <target>

# Can the source reach the target port?
docker run --rm --network <network> curlimages/curl:latest http://<target>:<port>

# Does the network have the expected subnet and endpoints?
docker network inspect <network>

# Is the host port published as expected?
docker port <container>
```

Common causes:

1. The containers are on different networks.
2. The application listens only on `127.0.0.1` instead of `0.0.0.0`.
3. The wrong container port or host port was used.
4. The port was not published with `-p`.
5. The Docker subnet overlaps the host, VPN, VPC, or another route.
6. Host firewall, cloud security groups, or Docker firewall rules block traffic.
7. The service is unhealthy or has not started listening yet.
8. A network was marked `--internal` or a container was attached to the wrong network.

## Scenario-Based Interview Questions

### 1. Two containers cannot communicate. How do you debug it?

Check that both containers are running and attached to the same user-defined network. Inspect network membership, resolve the target by service name, test the target port, and verify that the application listens on `0.0.0.0`. Then inspect firewalls, policies, and subnet overlap.

### 2. A container can reach another container by IP but not by name. Why?

They may be on the legacy default bridge, on different networks, or using an incorrect alias. Attach both containers to the same user-defined bridge and use Docker's embedded DNS with the service name.

### 3. The application works inside the container but not from a browser. What do you check?

Check that the container port is published with `-p`, the host port is correct, the application listens on `0.0.0.0`, and host/cloud firewalls permit traffic. `EXPOSE` alone does not publish a port.

### 4. Why can a container not be reached from another Docker network?

Docker networks are separate logical networks. Attach the source container to the destination network or connect the destination to a deliberately shared network. Do not route around the design by hard-coding private IPs.

### 5. Why is a custom bridge preferred over the default bridge?

It provides automatic DNS, explicit membership, network aliases, runtime connect/disconnect, and better isolation between application stacks. It also makes the network design visible in configuration and automation.

### 6. What is the difference between `EXPOSE` and `-p`?

`EXPOSE` documents a container port in image metadata. `-p` publishes and forwards a host port to a container port. Only the latter makes the service reachable through the host's published port.

### 7. Why should a database not be published publicly?

It increases attack surface and bypasses the intended application boundary. Put it on an internal backend network, allow only the application service to reach it, and enforce database authentication and encryption.

### 8. What happens when a container uses host networking?

It shares the host network namespace, so it has no separate container IP and no normal bridge/veth isolation. It can bind host ports directly, which can improve performance but creates port conflicts and a larger security impact.

### 9. What does `172.18.0.2/16` mean?

It means the container has address `172.18.0.2` in a network whose first 16 bits identify the network. The subnet mask is `255.255.0.0`, and Docker uses the configured bridge gateway and IPAM pool to route traffic.

### 10. What happens if Docker's veth interface is deleted?

The container's endpoint is disconnected from the bridge and networking can fail. Check the container and network state, then allow Docker to recreate the endpoint by reconnecting the network or recreating the container. Do not manually delete Docker interfaces as a routine fix.

### 11. Can containers on the same bridge access each other's data?

Networking alone does not expose filesystem data. A compromised service can attempt to connect to network services, however. Protect each service with authentication, authorization, TLS, least-privilege network membership, and separate networks where appropriate.

### 12. What is the risk of overlapping Docker and corporate CIDRs?

The host may choose the wrong route because the same destination appears on multiple interfaces. This causes intermittent or complete failures, especially with VPNs and cloud VPCs. Allocate non-overlapping private ranges and document them.

### 13. How would you connect containers across multiple hosts?

Use an orchestrator-supported overlay or cloud-native network, configure the required control and data plane connectivity, and define service discovery and encryption. A local bridge network cannot directly connect containers on different Docker hosts.

### 14. A service was recreated and its IP changed. Will clients break?

Clients that hard-code the old IP can break. Clients using a user-defined network's service name resolve the current endpoint and are more resilient to replacement.

### 15. How do you isolate login, finance, and database services?

Use separate frontend and backend networks. Put login on frontend, database on backend, and finance on both. Publish only login, keep database unpublished, and enforce authentication and authorization between services.

## Final Interview Checklist

- A container normally has its own network namespace and virtual `eth0`.
- A veth pair connects the container namespace to a host bridge.
- The bridge driver is local to one host; overlay is for multi-host networking.
- User-defined bridge networks provide DNS and explicit membership.
- `EXPOSE` is metadata; `-p` publishes a host port.
- Host networking removes network namespace isolation; it is not the default security choice.
- CIDR defines the subnet and must not overlap host, VPN, VPC, or LAN routes.
- Use service names instead of container IPs.
- Keep databases on internal networks and publish only required ingress services.
- Network isolation reduces reachability; it does not replace authentication, TLS, authorization, or firewalls.
