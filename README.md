# Docker Networking and Security

This repository provides a concise overview of **Docker Networking** modes and essential **Docker Security** best practices. It's aimed at developers, DevOps engineers, and security professionals looking to strengthen container environments.

---

## Docker Networking Modes

Docker provides several networking drivers that determine how containers communicate with each other and the external world.

### 1. Default Bridge

The default `bridge` network is created automatically by Docker. Containers on this network can communicate with each other using their IP addresses but not via DNS by default. It provides basic isolation between the host and container network spaces.

* **Use Case:** Simple standalone container deployments.
* **Limitations:** No name resolution, limited control over network configuration.
* **Example:**

```bash
docker run -d --name container1 nginx
```

### 2. User-Defined Bridge

A user-defined bridge allows more control and better container isolation. It supports DNS-based container name resolution and custom subnetting.

* **Use Case:** Multi-container applications requiring container name-based discovery.
* **Benefits:** Better isolation, name resolution, easier debugging.
* **Example:**

```bash
docker network create my-bridge

docker run -d --name app1 --network my-bridge nginx

docker run -d --name app2 --network my-bridge alpine sleep 1000
```

### 3. Host Network

The `host` network mode removes the network namespace isolation between the container and host. The container uses the host’s network stack directly.

* **Use Case:** Applications requiring high network performance or port binding conflicts.
* **Limitations:** No network isolation; security risks due to shared namespace.
* **Example:**

```bash
docker run --rm --network host nginx
```

### 4. Macvlan & Macvlan 802.1Q

`macvlan` allows containers to have their own MAC addresses and appear as physical devices on the network. The `802.1Q` mode adds VLAN tagging support for segmenting traffic into VLANs.

* **Use Case:** Direct communication with physical network, legacy systems integration.
* **Benefits:** MAC-level separation, VLAN-aware container networking.
* **Example:**

```bash
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 macvlan_net

docker run --rm --net=macvlan_net busybox ifconfig
```

### 5. IPvlan

Similar to macvlan but works by assigning IPs directly from the parent interface. It reduces MAC address overhead and can be more efficient.

* **Use Case:** IP routing without MAC-based communication.
* **Benefits:** Simpler MAC layer, better performance on certain switches.
* **Example:**

```bash
docker network create -d ipvlan \
  --subnet=192.168.2.0/24 \
  --gateway=192.168.2.1 \
  -o parent=eth0 ipvlan_net

docker run --rm --net=ipvlan_net alpine ip a
```

### 6. IPvlan L3 Mode

In L3 mode, IPvlan uses IP routing instead of bridging. This allows containers to communicate across different subnets without MAC address requirements.

* **Use Case:** Cross-subnet communication in routed networks.
* **Benefits:** Lighter footprint, scalable routing.
* **Example:**

```bash
docker network create -d ipvlan \
  --subnet=192.168.3.0/24 \
  --gateway=192.168.3.1 \
  -o ipvlan_mode=l3 \
  -o parent=eth0 ipvlan_l3_net

docker run --rm --net=ipvlan_l3_net alpine ip route
```

### 7. None

This disables all networking for the container. The container has no access to other containers or external networks.

* **Use Case:** Highly isolated workloads, testing, or tasks that don’t require network access.
* **Example:**

```bash
docker run --rm --network none alpine ping 8.8.8.8
# Output: ping: bad address '8.8.8.8'
```

---

## Docker Security Best Practices

### 1. Run Containers as Unprivileged Users

* Avoid running containers as `root`.
* Use `USER` directive in Dockerfile.

```Dockerfile
FROM alpine
RUN adduser -D appuser
USER appuser
```

### 2. Disable Root User

* Use rootless Docker if possible.
* Prevent access to sensitive host files.

### 3. Prevent Privilege Escalation

* Use `--security-opt "no-new-privileges:true"` to avoid acquiring extra permissions.

```bash
docker run --security-opt "no-new-privileges:true" myimage
```

### 4. Limit Container Capabilities

* Drop unnecessary Linux capabilities with `--cap-drop`.

```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myimage
```

### 5. Filesystem Permissions and Access

* Use `read-only` file systems where possible.
* Limit writable volumes.

```bash
docker run --read-only -v /tmp:/tmp myimage
```

### 6. Disable Inter-Container Communication

* Set `--icc=false` in Docker daemon configuration to prevent containers from talking to each other.

```json
{
  "icc": false
}
```
