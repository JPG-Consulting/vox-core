# PROJECT_OVERVIEW.md

## Purpose

This document provides a **high-level overview and authoritative index** of the project.

It defines:
- what the project is and is not
- core design principles
- documentation structure and dependencies
- the intended way humans and AI systems should understand the project

This document is the **single entry point** to the documentation set.
All contributors (human or AI) MUST read this document first.

---

## Project Summary

This project implements a **distributed, voice-first AI assistant backend**
designed to work with lightweight client devices (“satellites”) such as
ESP32-S3 or Raspberry Pi Zero 2W.

Key characteristics:
- low-latency, streaming voice interaction (STT → LLM → TTS)
- multi-user conversations with identity awareness
- strict privacy, consent, and security guarantees
- role-based command execution with delegation and confirmation
- extensible design (vision, RAG, external LLM fallback)

The system is designed to run on **MiniPC-class hardware** by enforcing
conservative concurrency, explicit state machines, and fail-closed behavior.

---

## Core Design Principles

### 1) Security and Privacy by Design

- Identity is probabilistic and stateful.
- Private data is never disclosed by default.
- Consent is explicit, scoped, and revocable.
- Commands require authorization and, when risky, confirmation.

If behavior is ambiguous, the system MUST choose the safest option.

---

### 2) Separation of Concerns

The system is divided into:
- **Core / Orchestrator**: state, policy, enforcement
- **Providers**: STT, TTS, LLM, Vision (pluggable, replaceable)
- **Satellites**: audio/image capture and playback

The LLM is treated as a **text generator**, not a policy engine.

---

### 3) Explicit State Machines

All non-trivial behavior is governed by explicit state machines:
- identity resolution and locking
- conversation modes
- command execution and confirmation
- permission delegation
- consent lifecycle

No implicit state transitions are allowed.

---

### 4) Designed for AI-Assisted Development

This repository is explicitly designed to be developed with AI tools.

Documentation:
- uses normative language (MUST / MUST NOT / SHOULD)
- declares dependencies between documents
- is authoritative and versioned
- must stay consistent with code

---

## System Scope

### In Scope
- voice-based interaction
- multi-user identity handling
- conversation vs profile memory separation
- role-based commands and delegation
- confirmation-required risky actions
- optional vision input (face detection / identification)
- optional RAG and external LLM fallback

### Out of Scope (for now)
- large-scale cloud deployment
- biometric-grade identity guarantees
- autonomous web crawling
- end-user mobile applications

---

## Document Map (Authoritative)

The following documents define the project.
Dependencies are explicit and MUST be respected.

### Entry & Collaboration

- **AI_COLLABORATION.md**  
  Rules for how AI assistants must work with this repository.

---

### Security & Rules

- **SECURITY_INVARIANTS.md**  
  Non-negotiable rules that MUST NEVER be violated.  
  Overrides all other documents in case of conflict.

---

### Architecture & Deployment

- **ARCHITECTURE.md**  
  System components, boundaries, and data flow.  
  Depends on: PROJECT_OVERVIEW.md

- **DEPLOYMENT.md**  
  Deployment guidelines for MiniPC-class hardware.  
  Depends on: ARCHITECTURE.md, SECURITY_INVARIANTS.md

---

### State & Data

- **STATE_MACHINES.md**  
  All state machines (identity, conversation, commands, consent).  
  Depends on: ARCHITECTURE.md, SECURITY_INVARIANTS.md

- **DATA_MODELS.md**  
  Schemas for users, conversations, permissions, delegations, pending objects.  
  Used by: STATE_MACHINES.md, COMMANDS.md, RAG.md

---

### Intents & Commands

- **INTENTS.md**  
  Definition and categorization of intents.

- **INTENTS_COMMAND_MAPPING.md**  
  Authoritative mapping of which intents may execute which commands.  
  Depends on: INTENTS.md, COMMANDS.md, SECURITY_INVARIANTS.md

- **COMMANDS.md**  
  Command taxonomy, authorization, delegation, confirmation rules.  
  Depends on: SECURITY_INVARIANTS.md, STATE_MACHINES.md, DATA_MODELS.md

---

### Identity & Perception

- **IDENTITY_FUSION.md**  
  Fusion rules for voice, vision, satellite, and explicit identity signals.  
  Depends on: STATE_MACHINES.md, DATA_MODELS.md, SECURITY_INVARIANTS.md

- **VISION.md**  
  Image handling, face detection, and identity signals from vision.  
  Depends on: IDENTITY_FUSION.md, SECURITY_INVARIANTS.md

---

### Knowledge & Reasoning

- **RAG.md**  
  Retrieval-Augmented Generation strategy and external knowledge fallback.  
  Depends on: INTENTS.md, DATA_MODELS.md, SECURITY_INVARIANTS.md

---


---

### Roadmap (Non-Authoritative)

- **TODO.md**  
  Project roadmap and open tasks.  
  **Non-authoritative**: this file is not a specification and MUST NOT override
  any document under `docs/`.  
  Intended for planning and coordination only.


## Intended Reading Order

For humans and AI assistants:

1. PROJECT_OVERVIEW.md
2. AI_COLLABORATION.md
3. SECURITY_INVARIANTS.md
4. ARCHITECTURE.md
5. DEPLOYMENT.md
6. STATE_MACHINES.md
7. DATA_MODELS.md
8. INTENTS.md
9. INTENTS_COMMAND_MAPPING.md
10. COMMANDS.md
11. IDENTITY_FUSION.md
12. VISION.md
13. RAG.md

---

## Update Rules

- If a document change affects another, both MUST be updated.
- Behavior changes require documentation updates.
- SECURITY_INVARIANTS.md always has priority.
- PROJECT_OVERVIEW.md must remain accurate.

---

## Change Log

- 2026-01-16: Updated to include identity fusion, intent-command mapping,
  vision, RAG, and deployment documentation.
