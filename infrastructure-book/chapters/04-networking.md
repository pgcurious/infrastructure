# Chapter 4: Network Fundamentals for Infrastructure

*Problem: You have millions of computers that need to communicate. How do you identify each one? How do you route messages across the world? How do you handle thousands of connections hitting one server?*

---

## The Language of Networks

Networking can feel overwhelming—layers upon layers of protocols, acronyms everywhere. But for infrastructure work, you need to deeply understand just a few key concepts. Let's build that foundation.

## IP Addresses: The Location System

An **IP address** is a numerical identifier for a device on a network. Think of it as a street address for computers.

### IPv4: The Original

IPv4 addresses look like: `192.168.1.100`

That's four numbers (0-255), separated by dots. Each number is one byte, so an IPv4 address is 32 bits total.

```
192.168.1.100

Each section = 8 bits (0-255)
Total = 32 bits = ~4.3 billion possible addresses
```

This seemed like plenty in the 1980s. It wasn't.

### IPv6: The Future (That's Slowly Arriving)

IPv6 addresses look like: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`

That's 128 bits—340 undecillion addresses (340 followed by 36 zeros). Enough for every grain of sand on Earth to have trillions of addresses.

For infrastructure work, you'll mostly deal with IPv4 still, but IPv6 is increasingly important.

### Public vs. Private IP Addresses

Not all IP addresses are visible on the internet.

**Private IP ranges** (used inside organizations):
- `10.0.0.0` to `10.255.255.255`
- `172.16.0.0` to `172.31.255.255`
- `192.168.0.0` to `192.168.255.255`

Your laptop at home probably has an IP like `192.168.1.x`. This address only works inside your home network.

**Public IP addresses** are unique across the entire internet. When you make a request to a website, your router performs **NAT (Network Address Translation)** to translate your private address to a public one.

```
Your Laptop (192.168.1.100)
         │
         ▼
Your Router (192.168.1.1 internal, 98.45.123.77 public)
         │
         ▼ NAT translates
Internet sees: Request from 98.45.123.77
```

> **Think About It:** Why do cloud providers charge for public IP addresses but not private ones? What's the scarcity here?

## Ports: The Apartment Numbers

An IP address gets you to the machine. A **port** gets you to the specific application on that machine.

In our city analogy: if the IP address is the building address, the port is the apartment number. Multiple tenants (applications) can live at the same address (IP), each with their own door (port).

### Port Numbers

Ports are 16-bit numbers, ranging from **0 to 65,535**.

They're divided into ranges:
- **0-1023**: Well-known ports (require root/admin to bind)
- **1024-49151**: Registered ports (applications can register these)
- **49152-65535**: Dynamic/ephemeral ports (used for outgoing connections)

### Common Ports You'll See

| Port | Protocol | Use |
|------|----------|-----|
| 22 | SSH | Secure remote terminal |
| 80 | HTTP | Web traffic (unencrypted) |
| 443 | HTTPS | Web traffic (encrypted) |
| 3306 | MySQL | MySQL database |
| 5432 | PostgreSQL | PostgreSQL database |
| 6379 | Redis | Redis cache |
| 8080 | Alt HTTP | Common for development/proxies |
| 27017 | MongoDB | MongoDB database |

### The Socket: IP + Port

When your application opens a network connection, it creates a **socket**—a combination of IP address and port. A connection is uniquely identified by:

```
Source IP:Source Port ←→ Destination IP:Destination Port

Example:
192.168.1.100:54321 ←→ 93.184.216.34:443
(Your laptop)             (example.com HTTPS)
```

This is why you can have multiple browser tabs open to the same website—each uses a different source port.

### Ports in Java/Spring Boot

When you start a Spring Boot application:

```java
// application.properties
server.port=8080
```

Your application binds to port 8080. Incoming requests to `your-ip:8080` reach your application.

```bash
# Check what's listening on ports (Linux)
netstat -tlnp
# or
ss -tlnp

# Output:
# LISTEN  0  128  0.0.0.0:8080  0.0.0.0:*  users:(("java",pid=12345))
```

## DNS: The Phone Book

Remembering `93.184.216.34` is inconvenient. Humans prefer `example.com`. The **Domain Name System (DNS)** translates human-readable names to IP addresses.

### How DNS Resolution Works

When you type `api.mycompany.com` in your browser:

```
1. Browser: "What's the IP for api.mycompany.com?"
           │
           ▼
2. Local DNS Cache → Found? Return cached IP
           │ Not found
           ▼
3. Recursive DNS Resolver (usually your ISP's)
           │
           ▼ "Ask the root servers"
4. Root DNS Server: "Try the .com servers"
           │
           ▼
5. .com TLD Server: "Try mycompany.com's nameservers"
           │
           ▼
6. mycompany.com Nameserver: "api.mycompany.com is 10.0.1.50"
           │
           ▼
7. Answer cached at each level, returned to browser
```

### DNS Record Types

| Type | Purpose | Example |
|------|---------|---------|
| **A** | Maps name to IPv4 address | `api.example.com → 93.184.216.34` |
| **AAAA** | Maps name to IPv6 address | `api.example.com → 2001:db8::1` |
| **CNAME** | Alias to another name | `www.example.com → example.com` |
| **MX** | Mail server for domain | `example.com mail → mail.example.com` |
| **TXT** | Arbitrary text (verification, SPF) | Used for domain verification |

### TTL: Time To Live

Each DNS record has a **TTL**—how long resolvers should cache the answer.

- **High TTL (e.g., 86400 = 24 hours)**: Less DNS traffic, but changes propagate slowly
- **Low TTL (e.g., 60 = 1 minute)**: Changes propagate quickly, but more DNS queries

When preparing for a migration, you often lower TTLs in advance so the switch happens quickly.

### DNS in Infrastructure

DNS is critical for:

**Service discovery**: Internal services find each other via DNS names, not hardcoded IPs
```properties
# application.properties
spring.datasource.url=jdbc:postgresql://db.internal.mycompany.com:5432/mydb
```

**Load balancing**: DNS can return multiple IPs (round-robin DNS)

**Failover**: Change DNS to point to a backup system during outages

> **Think About It:** If DNS is so important, what happens if your DNS provider goes down? How would you make DNS resilient?

## Load Balancers: Traffic Distribution

What happens when one server can't handle all the traffic? You add more servers. But how do clients know which server to connect to?

A **load balancer** sits in front of multiple servers and distributes incoming requests across them.

```
                    Internet
                        │
                        ▼
                 ┌──────────────┐
                 │    Load      │  Public IP: 203.0.113.50
                 │   Balancer   │
                 └──────┬───────┘
                        │
           ┌────────────┼────────────┐
           │            │            │
           ▼            ▼            ▼
      ┌────────┐   ┌────────┐   ┌────────┐
      │Server 1│   │Server 2│   │Server 3│
      │10.0.1.1│   │10.0.1.2│   │10.0.1.3│
      └────────┘   └────────┘   └────────┘
```

### Load Balancing Algorithms

**Round Robin**: Each request goes to the next server in rotation. Simple, but ignores server capacity.

**Least Connections**: Send to the server with fewest active connections. Good when requests have variable processing times.

**IP Hash**: Hash the client IP to pick a server. Same client always goes to same server (useful for session affinity).

**Weighted**: Assign weights to servers based on capacity. A server with weight 3 gets triple the traffic of weight 1.

### Layer 4 vs. Layer 7 Load Balancing

**Layer 4 (Transport)**: Balances based on IP and port. Fast, but can't inspect request content.

**Layer 7 (Application)**: Balances based on HTTP headers, URL paths, cookies. More flexible, slightly more overhead.

```
Layer 7 example:
- /api/* → Backend API servers
- /static/* → Static content servers
- /admin/* → Admin server (with extra authentication)
```

### Health Checks

Load balancers continuously check if servers are healthy:

```yaml
# Example health check configuration
health_check:
  path: /actuator/health
  interval: 10s
  timeout: 5s
  unhealthy_threshold: 3
  healthy_threshold: 2
```

If a server fails 3 consecutive checks, the load balancer stops sending traffic to it. When it recovers, traffic resumes.

In Spring Boot, you'd expose a health endpoint:

```java
// Automatically provided by Spring Boot Actuator
// GET /actuator/health
// Returns: {"status": "UP"}
```

## Putting It Together: A Request's Network Journey

Let's trace a request from browser to server:

```
1. User types: https://api.myapp.com/users

2. Browser checks DNS cache → miss

3. DNS resolution:
   api.myapp.com → 203.0.113.50 (load balancer IP)

4. TCP connection:
   Browser (192.168.1.100:54321) → 203.0.113.50:443
   Three-way handshake (SYN, SYN-ACK, ACK)

5. TLS handshake:
   Negotiate encryption, verify certificate

6. HTTP request over encrypted connection:
   GET /users HTTP/2
   Host: api.myapp.com

7. Load balancer receives on :443
   Selects Server 2 (10.0.1.2) based on least connections
   Forwards to 10.0.1.2:8080

8. Server 2 processes request
   Responds with JSON

9. Response flows back:
   Server 2 → Load Balancer → Browser

10. Browser renders response
```

All of this happens in milliseconds.

## Common Network Issues You'll Debug

**Connection refused**: Nothing is listening on that IP:port. Server not running? Wrong port?

**Connection timeout**: Packets aren't reaching the destination. Firewall blocking? Wrong IP? Network outage?

**DNS resolution failed**: Can't translate hostname to IP. DNS server down? Typo in hostname?

**Port already in use**: Another process has bound to that port. Find it with `netstat` or `ss`.

```bash
# Find what's using port 8080
lsof -i :8080

# Or
ss -tlnp | grep 8080
```

---

## Key Takeaways

1. **IP addresses identify machines, ports identify applications** - Together they form sockets that uniquely identify network connections. Private IPs work inside networks; public IPs are visible on the internet.

2. **DNS is the internet's phone book—and a critical dependency** - It translates human-readable names to IP addresses. Understand TTL for managing changes, and always have DNS redundancy.

3. **Load balancers distribute traffic and enable scaling** - They hide multiple servers behind a single IP, perform health checks, and route traffic intelligently. Most production systems need load balancing.

---

*Next Chapter: [The Container Revolution](05-containers.md) - Lightweight isolation that changed how we deploy software*
