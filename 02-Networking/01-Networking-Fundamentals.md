# 🌐 Networking Fundamentals for DevOps


## 🖼️ Quick Visual Summary

![Quick Summary: Networking Fundamentals for DevOps](../assets/topic-summaries/networking-fundamentals-devops.svg)

> **⚡ 80/20 Summary:** DNS finds IPs • ports find apps • firewalls allow traffic • load balancers spread requests

## 1. 🎯 Overview
Networking connects users, servers, containers, Kubernetes clusters, cloud services, and CI/CD systems. DevOps engineers do not need to be network architects first, but they must debug connectivity, DNS, ports, firewalls, latency, and load balancing quickly.

## 2. 💡 Why This Matters
- **Most production issues look like networking first:** timeout, connection refused, DNS failure, TLS error, slow response.
- **Cloud and Kubernetes are network-heavy:** VPCs, subnets, services, ingress, security groups, and load balancers are daily work.
- **Good debugging saves hours:** knowing where traffic stops is more valuable than memorizing every protocol.

## 3. 🧠 80/20 Core Concepts
- **IP address:** unique address of a machine or network interface.
- **Subnet:** a smaller range of IPs inside a network, usually used to separate public/private resources.
- **Port:** application door on a machine, such as `22` for SSH, `80` for HTTP, `443` for HTTPS.
- **Protocol:** rules for communication, mainly TCP, UDP, HTTP, HTTPS, SSH, DNS.
- **DNS:** converts names like `api.example.com` into IP addresses.
- **Firewall / Security Group:** allows or blocks traffic based on source, destination, protocol, and port.
- **Routing:** decides where packets go next.
- **Load Balancer:** spreads traffic across healthy backend servers or pods.

## 4. 🧭 Request Flow
```text
User
  -> DNS lookup
  -> Public IP / Load Balancer
  -> Firewall / Security Group
  -> Web Server / Ingress
  -> Application
  -> Database / Cache / Internal Service
```

## 5. 🛠️ Commands & Practical Usage

Check if a host is reachable:
```bash
ping google.com
```

Check DNS resolution:
```bash
dig example.com
nslookup example.com
```

Test an HTTP endpoint:
```bash
curl -I https://example.com
curl -v https://example.com
```

Check listening ports:
```bash
ss -tulnp
```

Trace the network path:
```bash
traceroute example.com
```

## 6. 🚨 Common Errors & Fixes
- **DNS not resolving:** check DNS record, nameserver, typo, or expired domain.
- **Connection refused:** server is reachable, but no process is listening on that port.
- **Connection timed out:** traffic is blocked, route is broken, or target is down.
- **TLS certificate error:** certificate expired, wrong hostname, or incomplete chain.
- **Works locally but not in cloud:** check security group, subnet route table, NACL, and application bind address.

## 7. 🎤 Interview Questions

**Q1: What is the difference between TCP and UDP?**  
TCP is connection-oriented and reliable. UDP is faster but does not guarantee delivery.

**Q2: What happens when you open `https://example.com`?**  
DNS resolves the domain, the client connects to the server on port `443`, TLS handshake happens, then HTTP request/response flows.

**Q3: What does `connection refused` mean?**  
The host is reachable, but the target port has no service listening or actively rejected the request.

**Q4: What is a load balancer?**  
A component that distributes traffic across multiple healthy backends to improve availability and scalability.

## 8. ⚡ Quick Revision
- DNS gives the IP.
- Ports identify the application.
- Firewalls decide what is allowed.
- Routing decides where traffic goes.
- Load balancers distribute traffic.
- Use `curl`, `dig`, `ss`, and `traceroute` first while debugging.
