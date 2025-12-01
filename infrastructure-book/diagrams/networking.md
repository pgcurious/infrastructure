# Networking Diagrams

## DNS Resolution Flow

```mermaid
sequenceDiagram
    participant Client
    participant LocalCache as Local DNS Cache
    participant Resolver as DNS Resolver
    participant Root as Root DNS
    participant TLD as .com TLD
    participant Auth as example.com Nameserver

    Client->>LocalCache: What's api.example.com?
    LocalCache-->>Client: Not cached

    Client->>Resolver: What's api.example.com?
    Resolver->>Root: What's api.example.com?
    Root-->>Resolver: Ask .com servers

    Resolver->>TLD: What's api.example.com?
    TLD-->>Resolver: Ask example.com nameservers

    Resolver->>Auth: What's api.example.com?
    Auth-->>Resolver: 93.184.216.34 (TTL: 300s)

    Resolver-->>Client: 93.184.216.34
    Note over Client,LocalCache: Result cached for 300 seconds
```

## Load Balancer Patterns

```mermaid
graph TB
    subgraph "Round Robin"
        LB1[Load Balancer]
        S1A[Server 1]
        S1B[Server 2]
        S1C[Server 3]

        LB1 -->|Request 1| S1A
        LB1 -->|Request 2| S1B
        LB1 -->|Request 3| S1C
        LB1 -->|Request 4| S1A
    end
```

```mermaid
graph TB
    subgraph "Least Connections"
        LB2[Load Balancer]
        S2A[Server 1: 5 connections]
        S2B[Server 2: 2 connections]
        S2C[Server 3: 8 connections]

        LB2 -->|New Request| S2B
    end
```

## Port Mapping

```
┌──────────────────────────────────────────────────────────────┐
│                        Host Machine                           │
│                     Public IP: 203.0.113.50                  │
│                                                              │
│  Port 80 ──────────────────────┬─────────────────────────┐  │
│  Port 443 ─────────────────────┼──────────────────────┐  │  │
│  Port 8080 ────────────────────┼───────────────────┐  │  │  │
│                                │                   │  │  │  │
│  ┌─────────────────────────────┼───────────────────┼──┼──┼──┤
│  │         Docker Engine       │                   │  │  │  │
│  │                             │                   │  │  │  │
│  │  ┌─────────────────────┐    │    ┌────────────────────┐  │
│  │  │    Container A      │◄───┘    │    Container B     │  │
│  │  │    (nginx)          │         │    (api)           │  │
│  │  │    Port 80 ─────────┼─────────►    Port 8080       │  │
│  │  └─────────────────────┘         └────────────────────┘  │
│  │                                                          │
│  │  ┌─────────────────────┐                                 │
│  │  │    Container C      │◄────────── Port 443            │
│  │  │    (https-proxy)    │                                 │
│  │  │    Port 443         │                                 │
│  │  └─────────────────────┘                                 │
│  └──────────────────────────────────────────────────────────┤
└──────────────────────────────────────────────────────────────┘

External Access:
  http://203.0.113.50:80     → Container A (nginx)
  https://203.0.113.50:443   → Container C (https-proxy)
  http://203.0.113.50:8080   → Container B (api)
```

## TCP Three-Way Handshake

```mermaid
sequenceDiagram
    participant Client
    participant Server

    Note over Client,Server: Connection Establishment
    Client->>Server: SYN (seq=100)
    Server->>Client: SYN-ACK (seq=300, ack=101)
    Client->>Server: ACK (seq=101, ack=301)

    Note over Client,Server: Connection Established
    Client->>Server: Data Transfer
    Server->>Client: ACK + Response

    Note over Client,Server: Connection Termination
    Client->>Server: FIN
    Server->>Client: ACK
    Server->>Client: FIN
    Client->>Server: ACK
```

## Kubernetes Service Networking

```mermaid
graph TB
    subgraph "Kubernetes Cluster"
        subgraph "Service: order-service"
            SVC[ClusterIP: 10.96.50.100:80]
        end

        subgraph "Node 1"
            POD1[order-pod-abc<br/>10.244.1.15:8080]
            POD2[order-pod-def<br/>10.244.1.16:8080]
        end

        subgraph "Node 2"
            POD3[order-pod-ghi<br/>10.244.2.20:8080]
        end

        KPROXY[kube-proxy<br/>iptables rules]
    end

    CLIENT[Other Pod] -->|order-service:80| SVC
    SVC --> KPROXY
    KPROXY -->|round-robin| POD1
    KPROXY -->|round-robin| POD2
    KPROXY -->|round-robin| POD3
```
