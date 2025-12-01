# From Bare Metal to Containers: A Developer's Guide to IT Infrastructure

> *Understanding what happens below your code*

## About This Book

You write applications. You deploy them somewhere. But what exactly is that "somewhere"? What happens between your `git push` and your users seeing a response?

This mini book takes you on a journey from the physical hardware humming in a data center to the containers orchestrated by Kubernetes. We won't just explain *what* these technologies are—we'll explore *why* they exist. Every layer of infrastructure was invented to solve a real problem, and understanding those problems makes the solutions click.

## Who This Book Is For

- **Developers** who want to understand infrastructure beyond "it just works"
- **Junior engineers** preparing for roles that touch DevOps or platform work
- **Anyone** who's ever wondered what a hypervisor actually does

## The City Analogy

Throughout this book, we'll use a consistent analogy:

| Infrastructure | City Equivalent |
|---------------|-----------------|
| Data Center | Industrial District |
| Server Rack | Building |
| Physical Server | Floor of a Building |
| Virtual Machine | Apartment |
| Container | Room |

Just like a city evolved to organize people and resources efficiently, IT infrastructure evolved to organize computing workloads.

## Table of Contents

1. **[The Physical Foundation](chapters/01-physical-foundation.md)** - Where computing actually happens
2. **[The Virtualization Revolution](chapters/02-virtualization.md)** - Why we stopped dedicating servers
3. **[Operating System Essentials](chapters/03-operating-system.md)** - The resource manager you take for granted
4. **[Network Fundamentals](chapters/04-networking.md)** - How computers find and talk to each other
5. **[The Container Revolution](chapters/05-containers.md)** - Lightweight isolation that changed everything
6. **[Orchestration](chapters/06-orchestration.md)** - Managing containers at scale
7. **[Putting It All Together](chapters/07-putting-it-together.md)** - A request's complete journey

**[Appendix: Quick Reference](appendix/quick-reference.md)** - Terms and definitions at a glance

## How to Read This Book

Each chapter builds on the previous one, so reading in order is recommended. However, if you're already familiar with certain concepts, feel free to skip ahead.

Every chapter follows a consistent structure:
- **Problem Statement** - What issue made humans invent this?
- **Core Concepts** - The essential knowledge
- **Think About It** - Reflective questions to deepen understanding
- **Key Takeaways** - Three things to remember

## What This Book Is Not

This is not a deep dive into any single technology. You won't learn how to configure a specific hypervisor or write Kubernetes manifests from scratch. Instead, you'll gain the mental model that makes learning those specifics much easier.

---

*Estimated reading time: 2-3 hours*

*Word count: ~7,500 words*
