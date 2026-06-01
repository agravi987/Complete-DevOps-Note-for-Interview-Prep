# 🧱 Ports, Firewalls, and Troubleshooting


## 🖼️ Quick Visual Summary

![Quick Summary: Ports, Firewalls, and Troubleshooting](../assets/topic-summaries/ports-firewalls-troubleshooting.svg)

> **⚡ 80/20 Summary:** Know source and destination • test the port • inspect firewall rules • confirm app logs

## 1. 🎯 Overview
DevOps troubleshooting often means answering one question: **where is the traffic blocked?** Use a simple layer-by-layer approach instead of guessing.

## 2. 🔌 Important Ports

| Port | Use |
| --- | --- |
| `22` | SSH |
| `53` | DNS |
| `80` | HTTP |
| `443` | HTTPS |
| `3306` | MySQL |
| `5432` | PostgreSQL |
| `6379` | Redis |
| `8080` | Common app/test port |
| `9090` | Prometheus |
| `3000` | Grafana / Node apps |

## 3. 🧱 Firewall Concepts
- **Inbound rule:** traffic coming into a server/service.
- **Outbound rule:** traffic leaving a server/service.
- **Source:** who is allowed to connect.
- **Destination:** where traffic is going.
- **CIDR:** IP range format, such as `10.0.0.0/16`.
- **Least privilege:** open only the required port from the required source.

## 4. 🧭 Debugging Flow
```text
1. Is the hostname correct?
2. Does DNS resolve?
3. Is the target IP reachable?
4. Is the port open?
5. Is the app listening?
6. Are firewall/security group rules correct?
7. Do application logs show the request?
```

## 5. 🛠️ Commands

Check open/listening ports:
```bash
ss -tulnp
```

Test TCP connectivity:
```bash
nc -vz example.com 443
```

Check HTTP response:
```bash
curl -I https://example.com
```

Check local firewall status:
```bash
sudo ufw status
sudo iptables -L -n -v
```

## 6. 🚨 Common DevOps Scenarios
- **App starts but is unreachable:** app may be bound to `127.0.0.1` instead of `0.0.0.0`.
- **Kubernetes Service not working:** selector may not match pod labels.
- **Cloud VM unreachable:** security group may not allow inbound traffic.
- **Database connection fails:** database firewall may not allow the app subnet.
- **Intermittent failures:** load balancer may be sending traffic to unhealthy targets.

## 7. 🎤 Interview Questions

**Q1: What is the difference between `connection refused` and `timeout`?**  
Refused means the target replied but the port is closed. Timeout means packets are blocked, dropped, or the target is unreachable.

**Q2: Why should databases not be public?**  
They should stay in private networks and accept traffic only from trusted app subnets or security groups.

**Q3: What does `0.0.0.0` mean for an application bind address?**  
The app listens on all network interfaces, making it reachable from outside the machine if firewall rules allow it.

## 8. ⚡ Quick Revision
- Check DNS first.
- Check port second.
- Check firewall third.
- Check app logs fourth.
- Always know the expected source, destination, protocol, and port.

