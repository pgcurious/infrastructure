# Chapter 3: Operating System Essentials

*Problem: A computer has finite resources—CPU, memory, disk, network. Multiple programs want to use these resources simultaneously. Without a referee, programs would overwrite each other's memory, fight for CPU time, and crash constantly. How do we bring order to this chaos?*

---

## The Operating System: A Resource Manager

Forget everything you think you know about operating systems. Strip away the desktop icons, the window managers, the app stores. At its core, an operating system does one thing: **it manages resources**.

The OS is the supreme arbiter of who gets what, when, and for how long. It provides:

1. **Process management** - Deciding which programs run and when
2. **Memory management** - Giving each program its own isolated memory space
3. **File system management** - Organizing persistent storage
4. **Device management** - Providing access to hardware
5. **Security enforcement** - Preventing programs from interfering with each other

In our city analogy, if the VM is an apartment, the OS is the building management company. It handles utilities, enforces rules, manages common spaces, and makes sure one tenant's problems don't affect everyone else.

## Kernel Space vs. User Space: The Great Divide

The most important architectural concept in operating systems is the separation between **kernel space** and **user space**.

```
┌────────────────────────────────────────────────┐
│                 User Space                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │  Your    │ │  Web     │ │ Database │ ...   │
│  │  App     │ │  Server  │ │  Server  │       │
│  └──────────┘ └──────────┘ └──────────┘       │
│                     │                          │
│              System Calls                      │
│                     │                          │
├─────────────────────▼──────────────────────────┤
│                Kernel Space                    │
│  ┌─────────────────────────────────────────┐  │
│  │  Process Scheduler │ Memory Manager     │  │
│  │  File System       │ Device Drivers     │  │
│  │  Network Stack     │ Security Module    │  │
│  └─────────────────────────────────────────┘  │
├────────────────────────────────────────────────┤
│              Physical Hardware                 │
│        CPU    RAM    Disk    Network          │
└────────────────────────────────────────────────┘
```

### The Kernel: Absolute Power

The **kernel** is the core of the operating system. It runs in **kernel space** with complete access to:
- All CPU instructions (including privileged ones)
- All physical memory
- All hardware devices

The kernel is trusted code. A bug in the kernel can crash the entire system or corrupt data. That's why kernels are relatively small and carefully maintained.

### User Space: The Sandbox

Your applications run in **user space**. They cannot:
- Directly access hardware
- Read another process's memory
- Execute privileged CPU instructions

When your Java application needs to read a file, it can't just talk to the disk. Instead:

1. Your app calls a library function (e.g., `Files.readAllBytes()` in Java)
2. The library makes a **system call** to the kernel
3. The kernel validates the request (Do you have permission? Does the file exist?)
4. If approved, the kernel reads from disk and copies data to your process
5. Control returns to your application with the data

This dance happens constantly—every file read, network packet, and screen update involves system calls.

> **Think About It:** Why is this separation important for security? What would happen if any program could directly access all hardware?

## Processes: Programs in Action

A **process** is a running instance of a program. The same program can have multiple processes—open three terminal windows, and you have three shell processes.

Each process gets:
- **Isolated memory space** - Process A cannot see or modify Process B's memory
- **A process ID (PID)** - A unique identifier
- **File descriptors** - Handles to open files, network connections, etc.
- **Environment variables** - Configuration passed from the parent process
- **A user/group ID** - For permission checking

### Process Lifecycle

```
    ┌─────────┐
    │  New    │  (process created)
    └────┬────┘
         │
         ▼
    ┌─────────┐    waiting for I/O    ┌─────────┐
    │  Ready  │ ◄──────────────────── │ Waiting │
    └────┬────┘                       └────▲────┘
         │                                 │
    gets CPU time                     needs I/O
         │                                 │
         ▼                                 │
    ┌─────────┐ ────────────────────────►──┘
    │ Running │
    └────┬────┘
         │
         ▼
    ┌─────────┐
    │ Exited  │  (process terminated)
    └─────────┘
```

### Process Creation

In Unix-like systems, processes are created through a `fork()` system call. The current process is cloned, creating a parent and child with identical memory (using copy-on-write for efficiency). The child typically then calls `exec()` to replace itself with a different program.

When you run `java -jar myapp.jar` in a terminal:
1. Your shell process calls `fork()`, creating a child
2. The child calls `exec()` to replace itself with the Java runtime
3. The Java runtime loads your JAR file and starts execution

### Viewing Processes

On Linux, you can see running processes:

```bash
# List all processes
ps aux

# Interactive process viewer
top

# Or the more modern alternative
htop
```

You'll see something like:
```
PID   USER    %CPU  %MEM  COMMAND
1     root    0.0   0.1   /sbin/init
245   root    0.0   0.2   /usr/sbin/sshd
1892  app     15.2  4.5   java -jar myapp.jar
```

## Threads: Lightweight Units of Execution

A **thread** is a unit of execution within a process. While processes are isolated from each other, threads within the same process share:
- Memory space (heap, global variables)
- File descriptors
- Process ID

But each thread has its own:
- Thread ID
- Stack (local variables, function call history)
- CPU register state
- Scheduling priority

### Why Threads Exist

Creating a process is expensive—the OS must set up a new memory space, copy file descriptors, initialize data structures. Threads are cheaper because they reuse most of the parent process's resources.

Consider a web server handling 1,000 concurrent connections:

**One process per connection**:
- 1,000 processes, each with ~10 MB overhead
- 10 GB of memory just for process overhead
- Context switching between processes is slow

**One thread per connection**:
- 1 process with 1,000 threads
- Shared memory space
- Faster context switching
- ~1 MB per thread stack

### Threads in Java

In Java, you work with threads regularly:

```java
// Creating a thread directly
Thread thread = new Thread(() -> {
    System.out.println("Running in thread: " + Thread.currentThread().getName());
});
thread.start();

// Using an ExecutorService (preferred)
ExecutorService executor = Executors.newFixedThreadPool(10);
executor.submit(() -> {
    // Your task here
});
```

Spring Boot's embedded Tomcat uses a thread pool to handle incoming HTTP requests. Each request is processed by a thread from the pool.

> **Think About It:** If threads share memory, what problems might occur when two threads try to modify the same data simultaneously?

## Memory Management: Every Byte Accounted For

The OS manages memory through **virtual memory**, giving each process the illusion of having its own contiguous address space.

```
Process A sees:              Process B sees:
┌─────────────────┐          ┌─────────────────┐
│ 0x0000 - Stack  │          │ 0x0000 - Stack  │
│      ...        │          │      ...        │
│ 0x4000 - Heap   │          │ 0x4000 - Heap   │
│      ...        │          │      ...        │
│ 0x8000 - Code   │          │ 0x8000 - Code   │
└─────────────────┘          └─────────────────┘

        │                            │
        └──────────┬─────────────────┘
                   │
                   ▼
            Physical RAM:
┌──────────────────────────────────────────────┐
│ A's Stack │ B's Code │ A's Heap │ B's Stack  │
└──────────────────────────────────────────────┘
```

The OS (with CPU hardware support) translates virtual addresses to physical addresses. Process A's address 0x4000 and Process B's address 0x4000 point to completely different physical memory.

### Memory Segments

A process's virtual memory is divided into segments:

| Segment | Contents |
|---------|----------|
| **Code/Text** | The executable instructions (read-only) |
| **Data** | Global and static variables |
| **Heap** | Dynamically allocated memory (grows upward) |
| **Stack** | Local variables, function call frames (grows downward) |

In Java, when you call `new Object()`, the JVM allocates memory from the heap. When you call a method, local variables go on the stack.

### Memory Limits and OOM

The OS tracks how much memory each process uses. When a process requests more memory than is available, the kernel's **OOM (Out of Memory) Killer** selects a process to terminate.

In infrastructure, you'll configure memory limits for applications:

```yaml
# Java JVM heap size
java -Xmx2g -Xms512m -jar myapp.jar

# Docker container memory limit
docker run --memory=2g myapp
```

## Why This Matters for Infrastructure

Understanding processes and threads directly impacts infrastructure decisions:

**Scaling strategies**:
- CPU-bound work benefits from more processes/threads (up to CPU core count)
- I/O-bound work can use many more threads (they're mostly waiting)

**Resource allocation**:
- How much memory does each process need?
- How many threads will your application spawn?
- What happens when limits are reached?

**Debugging production issues**:
- High CPU? Check which process/thread is consuming it
- Memory growing? Possible memory leak—which process?
- Application unresponsive? Maybe waiting on I/O or deadlocked threads

**Container design**:
- Containers share the host kernel (Chapter 5)
- Understanding kernel vs. user space explains container isolation

---

## Key Takeaways

1. **The OS is fundamentally a resource manager** - It arbitrates access to CPU, memory, disk, and network among competing processes. Understanding this helps you reason about application behavior and resource limits.

2. **Kernel/user space separation provides security and stability** - Applications can't directly access hardware or each other's memory. System calls are the controlled gateway between user code and kernel capabilities.

3. **Processes provide isolation, threads provide parallelism** - Processes have separate memory spaces and are heavier to create. Threads share memory and are lighter. Both have their place in application architecture.

---

*Next Chapter: [Network Fundamentals](04-networking.md) - How computers find and talk to each other*
