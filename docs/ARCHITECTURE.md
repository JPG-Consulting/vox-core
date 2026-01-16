# ARCHITECTURE.md

## Purpose

This document defines the **high-level architecture** of the system:
- main components
- process boundaries
- data flow between satellites and server
- responsibility separation

It describes **what talks to what** and **why**, not how individual rules are implemented.

This document MUST respect all rules defined in `SECURITY_INVARIANTS.md`.

---

## Architectural Overview

The system follows a **distributed client–server architecture**:

- **Satellites**  
  Lightweight client devices (ESP32-S3, Raspberry Pi Zero 2W, etc.)

- **Server**  
  Central backend responsible for orchestration, policy enforcement, and optional inference

- **Providers**  
  Pluggable implementations for STT, TTS, LLM, Vision, etc.

At a high level:

```
[ Satellite(s) ]
      |
      |  audio / text / image / identity signals
      v
[ Server Core / Orchestrator ]
      |
      +-- STT Provider
      +-- LLM Provider
      +-- TTS Provider
      +-- Vision Provider
      |
      v
[ Actions / Responses ]
```

---

## Core Architectural Principle

**The Server Core is the single source of truth for:**
- identity resolution
- privacy enforcement
- authorization
- command execution
- confirmation logic

Satellites and providers MUST NOT enforce security-critical rules.

---

## Components

### 1) Satellites

Satellites are client devices responsible for:
- capturing audio (microphone)
- optional local VAD / wake word
- optional local STT / TTS / face recognition
- playing back audio responses
- sending input streams to the server

Satellites MAY:
- pre-process audio (VAD, encoding)
- send identity signals if detected locally

Satellites MUST NOT:
- make authorization decisions
- decide command execution
- disclose private data autonomously

---

### 2) Server Core (Orchestrator)

The Server Core is the heart of the system.

Responsibilities:
- manage conversations
- resolve and lock identities
- enforce security invariants
- route data to providers
- merge identity signals (voice, image, satellite)
- manage conversation modes
- manage roles, permissions, delegation
- execute or deny commands
- filter context passed to the LLM

The Core MUST:
- be deterministic
- fail closed
- explicitly manage state

---

### 3) Providers

Providers are **replaceable modules** implementing a specific capability.

They MUST follow a common interface and MUST NOT enforce policy.

Provider types:
- STT Provider
- LLM Provider
- TTS Provider
- Vision Provider

Providers MAY be local or remote.

---

### 4) Policy and Guards (Logical Components)

Within the Core, the following logical subsystems exist:
- Identity Resolver
- Conversation Manager
- Privacy Guard
- Authorization Manager
- Command Execution Pipeline

---

## Data Flow (Voice Interaction)

1. Satellite captures audio
2. Satellite sends audio stream to server
3. Server optionally runs STT and speaker identification
4. Server updates conversation state
5. Server sends filtered context to LLM
6. LLM streams tokens back
7. Server feeds tokens into TTS
8. Audio is streamed back to satellite

Streaming is preferred at every step.

---

## Identity Signal Flow

Identity can be inferred from multiple sources:
- voice (speaker recognition)
- image (face recognition)
- satellite-provided identity
- explicit user statements

All identity signals are merged by the Core.

---

## Command Execution Flow

1. User speaks command
2. STT transcribes
3. Core detects intent
4. Core checks identity, role, delegation, conversation mode
5. Core checks command type (IMMEDIATE or CONFIRM_REQUIRED)
6. If confirmation required, a pending command is created
7. If authorized and confirmed, the command is executed
8. Otherwise, the command is denied safely

---

## Conversation and Memory Boundaries

The system maintains strict separation between:
- Conversation memory (shared within conversation)
- User profile memory (private by default)

The Core decides what context is visible to the LLM.

---

## Failure and Degradation Strategy

If a provider fails:
- the Core handles it gracefully
- unsafe actions are never executed

---

## Extensibility Guarantees

The architecture allows:
- adding new providers
- adding vision input later
- adding RAG and external LLM fallback

All extensions MUST respect `SECURITY_INVARIANTS.md`.

---

## Change Log

- 2026-01-16: Initial version.
