# DEPLOYMENT.md

## Purpose

This document defines **deployment guidelines** for the project, with a focus on:
- MiniPC-class hardware
- resource constraints
- process separation
- configuration and scaling strategy

This document does NOT define cloud-scale deployment.
It defines a **safe, realistic baseline** that works on modest hardware.

This document depends on:
- ARCHITECTURE.md
- SECURITY_INVARIANTS.md

---

## Target Hardware Profiles

### Supported Baseline Targets

#### MiniPC (Primary Target)
Examples:
- Intel N100 / N95 class MiniPC
- Nipogi AK1 Plus
- Similar low-power x86 systems

Typical specs:
- 4–8 CPU cores
- 8–16 GB RAM
- SSD storage
- No discrete GPU (optional iGPU)

---

#### Raspberry Pi 5 (Secondary Target)
- ARM64
- 8 GB RAM recommended
- Reduced concurrency

---

### Non-Goals
- Large multi-tenant servers
- High-availability clusters
- GPU-heavy inference

---

## Deployment Model

### Single-Node, Multi-Process

The recommended model is:

- **One server node**
- **Multiple processes**
- **Async I/O at the edges**

```text
[ WebSocket / HTTP Server ]
            |
            v
[ Core / Orchestrator ]
    |       |        |
    v       v        v
  STT     LLM      TTS
 Worker  Worker   Worker
```

Rationale:
- prevents blocking
- isolates failures
- simplifies resource control

---

## Process Responsibilities

### Core / Orchestrator
- Conversation management
- Identity fusion
- Privacy enforcement
- Authorization & confirmation
- Routing to workers

MUST be:
- fast
- deterministic
- non-blocking

---

### STT Worker
- streaming transcription
- speaker embeddings (optional)
- CPU-bound

Recommended:
- limit concurrency
- prioritize latency over throughput

---

### LLM Worker
- text generation
- tool calls
- streaming tokens

Recommended:
- single active generation at a time
- queue requests
- strict timeouts

---

### TTS Worker
- streaming audio synthesis
- voice caching
- CPU-bound

Recommended:
- allow parallel synthesis per conversation
- prewarm voices at startup

---

## Resource Management

### CPU
- STT and TTS may run in parallel
- LLM should be serialized
- Avoid oversubscription

---

### Memory
- Preload models at startup
- Avoid dynamic model swapping
- Cache small, reusable assets

---

### Disk
- Prefer SSD
- Store only:
  - models
  - logs
  - configuration
- Do NOT store raw audio or images by default

---

## Configuration Strategy

All configuration SHOULD be explicit and version-controlled.

Examples:
- enabled providers
- model sizes
- thresholds (identity, confidence)
- timeouts
- permissions

Environment variables MAY be used but SHOULD map to config files.

---

## Networking

Recommended:
- WebSocket for audio / streaming
- HTTP for control endpoints

Security:
- authentication between satellites and server
- local network only by default
- TLS if exposed externally

---

## Scaling Strategy

### Vertical Scaling (Primary)
- faster CPU
- more RAM
- better SSD

### Horizontal Scaling (Future)
- separate STT / TTS nodes
- shared message queue
- not required initially

---

## Failure and Recovery

- Workers MAY crash without stopping the Core
- Core MUST detect and recover
- Pending states MUST be cleared safely

On restart:
- conversations are lost
- profiles remain
- no unsafe action resumes automatically

---

## Observability

Recommended:
- structured logs
- minimal audit logs for commands
- latency metrics (TTFA, STT delay)

Avoid:
- verbose raw data logs
- storing sensitive content

---

## Deployment Checklist

Before production use:
- [ ] All SECURITY_INVARIANTS.md rules enforced
- [ ] Identity fusion tested
- [ ] Command confirmation tested
- [ ] Resource limits set
- [ ] Logs reviewed for privacy leaks

---

## Summary

- Designed for MiniPC-class hardware
- Single-node, multi-process model
- Conservative concurrency
- Explicit configuration
- Safe failure behavior

---

## Change Log

- 2026-01-16: Initial version.
