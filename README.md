````markdown
# Docker Networking and Security

This repository provides a practical and concise guide to **Docker Networking** modes and essential **Docker Security** best practices. It's designed for DevOps engineers, developers, and security enthusiasts aiming to deploy containers in a more robust and secure manner.

---

## 📡 Docker Networking Modes

Docker offers various networking drivers to manage how containers communicate with each other and the outside world.

### 1. Default Bridge
- Automatically created by Docker (`docker0`).
- Basic isolation, containers communicate via IP.
- No DNS-based discovery.

```bash
docker run -d --name container1 nginx
````

### 2. User-Defined Bridge

* Custom networks with DNS-based container name resolution.
* Better isolation and control.

```bash
docker network create my-bridge

docker run -d --name app1 --network my-bridge nginx
docker run -d --name app2 --network my-bridge alpine sleep 1000
```

### 3. Host Network

* Shares the host’s network stack.
* No network isolation between container and host.

```bash
docker run --rm --network host nginx
```

### 4. Macvlan & Macvlan 802.1Q

* Assigns MAC addresses to containers.
* VLAN tagging support with 802.1Q for segmented traffic.

```bash
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 macvlan_net

docker run --rm --net=macvlan_net busybox ifconfig
```

### 5. IPvlan

* Efficient alternative to macvlan.
* Direct IP assignment without extra MACs.

```bash
docker network create -d ipvlan \
  --subnet=192.168.2.0/24 \
  --gateway=192.168.2.1 \
  -o parent=eth0 ipvlan_net

docker run --rm --net=ipvlan_net alpine ip a
```

### 6. IPvlan L3

* Uses routing across subnets without MAC dependency.
* Ideal for routed environments.

```bash
docker network create -d ipvlan \
  --subnet=192.168.3.0/24 \
  --gateway=192.168.3.1 \
  -o ipvlan_mode=l3 \
  -o parent=eth0 ipvlan_l3_net

docker run --rm --net=ipvlan_l3_net alpine ip route
```

### 7. None

* Disables networking completely.
* Suitable for secure, standalone workloads.

```bash
docker run --rm --network none alpine ping 8.8.8.8
# Output: ping: bad address '8.8.8.8'
```

---

## 🔐 Docker Security Best Practices

### ✅ Run as Unprivileged User

Avoid root; declare a safer user in your Dockerfile:

```Dockerfile
FROM alpine
RUN adduser -D appuser
USER appuser
```

### ✅ Disable Root User (Use Rootless Docker)

Reduces attack surface and limits file system access.

### ✅ Prevent Privilege Escalation

```bash
docker run --security-opt "no-new-privileges:true" myimage
```

### ✅ Limit Container Capabilities

Drop unused capabilities; enable only what's needed.

```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myimage
```

### ✅ File System Access Controls

Use `read-only` mode and mount specific paths:

```bash
docker run --read-only -v /tmp:/tmp myimage
```

### ✅ Disable Inter-Container Communication

Add in `/etc/docker/daemon.json`:

```json
{
  "icc": false
}
```

---


```

---

Let me know if you’d like a `.gitignore`, LICENSE file, or GitHub repository setup instructions too.
```
