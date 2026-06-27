# 🔎 DNS, HTTP, and Load Balancing

## 🖼️ Quick Visual Summary

![Quick Summary: DNS, HTTP, and Load Balancing](../assets/topic-summaries/dns-http-load-balancing.svg)

> **80/20 Summary:** DNS finds the destination, HTTP carries the request, and load balancers share the work. 🚦

## 1. Big Picture

Ravi, this is the full request path in the real world.

Users do not type IP addresses every time.
They type names like `example.com`.
DNS turns that name into an IP.
Then HTTP or HTTPS carries the request.
Then the load balancer decides where the traffic should go.

## 2. Real-Life Analogy

Ravi, think of it like calling a restaurant 🍜

- **DNS** is the phone book
- **HTTP** is the conversation
- **HTTPS** is the secure conversation
- **Load balancer** is the host who sends you to the right table

## 3. Technical Definition

DNS maps names to IP addresses, HTTP defines request and response communication, and load balancing distributes traffic across healthy backend systems.

## 4. Internal Working

```text
Domain name
   |
   v
DNS resolves IP
   |
   v
Client sends HTTP/HTTPS request
   |
   v
Load balancer picks a healthy backend
   |
   v
Application responds
```

## 5. Key Concepts

| Concept | Meaning |
| --- | --- |
| A record | Maps a name to IPv4 address 🌍 |
| AAAA record | Maps a name to IPv6 address 🌍 |
| CNAME | Alias from one name to another 🔁 |
| TTL | How long DNS is cached ⏳ |
| GET | Read data 📖 |
| POST | Send new data ✉️ |
| 2xx | Success ✅ |
| 4xx | Client-side issue ⚠️ |
| 5xx | Server-side issue 💥 |
| L4 load balancer | Routes by IP and port |
| L7 load balancer | Understands HTTP paths and headers |

## 6. Commands

| Command | Why we use it | What happens internally |
| --- | --- | --- |
| `dig example.com` | Inspect DNS records | Queries DNS servers directly |
| `curl -I https://example.com` | See HTTP headers | Sends a request and prints response headers |
| `curl -v https://example.com` | Debug HTTP/TLS | Shows detailed connection and TLS info |
| `openssl s_client -connect example.com:443 -servername example.com` | Check TLS | Opens a secure socket and shows certificate details |

## 7. Real Production Usage

Ravi, this is where you see these concepts in production:

- a website resolves through DNS
- HTTPS secures user traffic
- a load balancer sends traffic to healthy app instances
- ingress controllers often sit in front of Kubernetes services

## 8. Common Mistakes

- ❌ Blaming the app before checking DNS
  - Why it is wrong: the app may be fine and the name may be wrong.
  - ✅ Correct: confirm resolution first.

- ❌ Forgetting HTTP status codes matter
  - Why it is wrong: they tell you where the failure happened.
  - ✅ Correct: use status codes as debugging clues.

- ❌ Ignoring TLS certificate issues
  - Why it is wrong: HTTPS can fail even when the app is healthy.
  - ✅ Correct: verify the certificate and hostname.

## 9. Best Practices

1. Always verify DNS before testing the app.
2. Use HTTPS in production.
3. Check load balancer health when traffic fails.
4. Learn common HTTP status codes.
5. Follow redirects carefully when debugging.

## 10. Interview Corner

Ravi, your interviewer might ask this. 🎤

**Q1: What does DNS do?**
A1: It maps domain names to IP addresses.

**Q2: What is HTTP?**
A2: A protocol for request and response communication.

**Q3: What is HTTPS?**
A3: HTTP secured with TLS encryption.

**Q4: What is a load balancer?**
A4: A system that distributes traffic across healthy backends.

**Q5: What does a 5xx status mean?**
A5: The server failed to process the request correctly.

## 11. Revision Summary

- DNS = name to IP 🔎
- HTTP = request and response 💬
- HTTPS = secure HTTP 🔒
- Load balancer = traffic manager 🚦
- Status codes = response hints 📟

## 12. Key Takeaways

- DNS is the first step in most requests.
- HTTP carries the conversation.
- HTTPS protects it.
- Load balancers help scale and survive failures.

## 13. Comparison Table

| HTTP | HTTPS | Load Balancer |
| --- | --- | --- |
| Plain request protocol | Encrypted request protocol | Traffic distributor |
| No encryption | Uses TLS | Shares load across backends |

## 14. Memory Tricks

- **DNS = directory**
- **HTTP = talk**
- **HTTPS = secure talk**
- **Load balancer = traffic helper**

## 15. Official Docs

- [MDN HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [Cloudflare DNS Guide](https://www.cloudflare.com/learning/dns/what-is-dns/)
