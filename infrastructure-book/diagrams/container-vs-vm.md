# Container vs Virtual Machine Architecture

## Virtual Machine Architecture

```mermaid
graph TB
    subgraph "VM Architecture"
        subgraph "VM 1"
            APP1[Application]
            BINS1[Bins/Libs]
            GOS1[Guest OS]
        end

        subgraph "VM 2"
            APP2[Application]
            BINS2[Bins/Libs]
            GOS2[Guest OS]
        end

        subgraph "VM 3"
            APP3[Application]
            BINS3[Bins/Libs]
            GOS3[Guest OS]
        end

        HYP[Hypervisor]
        HW[Physical Hardware]
    end

    APP1 --> BINS1
    BINS1 --> GOS1
    APP2 --> BINS2
    BINS2 --> GOS2
    APP3 --> BINS3
    BINS3 --> GOS3
    GOS1 --> HYP
    GOS2 --> HYP
    GOS3 --> HYP
    HYP --> HW
```

## Container Architecture

```mermaid
graph TB
    subgraph "Container Architecture"
        subgraph "Container 1"
            CAPP1[Application]
            CBINS1[Bins/Libs]
        end

        subgraph "Container 2"
            CAPP2[Application]
            CBINS2[Bins/Libs]
        end

        subgraph "Container 3"
            CAPP3[Application]
            CBINS3[Bins/Libs]
        end

        RUNTIME[Container Runtime]
        HOS[Host Operating System]
        HW2[Physical Hardware]
    end

    CAPP1 --> CBINS1
    CAPP2 --> CBINS2
    CAPP3 --> CBINS3
    CBINS1 --> RUNTIME
    CBINS2 --> RUNTIME
    CBINS3 --> RUNTIME
    RUNTIME --> HOS
    HOS --> HW2
```

## Comparison Table

| Aspect | Virtual Machines | Containers |
|--------|------------------|------------|
| Isolation | Hardware-level | OS-level |
| Size | GBs (includes OS) | MBs (no OS) |
| Boot Time | Minutes | Seconds |
| Resource Overhead | High | Low |
| OS Support | Any OS | Host kernel only |
| Security | Stronger isolation | Process isolation |

## Container Image Layers

```mermaid
graph TB
    subgraph "Container Image Layers"
        L1[Base Image: alpine:3.18]
        L2[Install JRE: eclipse-temurin:17-jre]
        L3[Copy Application: app.jar]
        L4[Set Entry Point: java -jar]
        RW[Read-Write Layer: Container Changes]
    end

    L1 --> L2
    L2 --> L3
    L3 --> L4
    L4 -.-> RW

    style L1 fill:#e1f5fe
    style L2 fill:#b3e5fc
    style L3 fill:#81d4fa
    style L4 fill:#4fc3f7
    style RW fill:#ffecb3
```
