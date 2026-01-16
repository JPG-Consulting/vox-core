# Vox Core

## Overview

**Vox Core** is a **privacy-first, voice-based AI assistant core** designed to run on modest hardware and coordinate lightweight client devices (“satellites”) such as ESP32-S3 or Raspberry Pi Zero 2W.

It provides a secure, state-driven backend for building voice assistants with:
- low-latency, streaming interaction (STT → LLM → TTS)
- multi-user identity awareness
- strict privacy, consent, and authorization rules
- role-based command execution with confirmation
- extensibility for vision and retrieval-augmented generation (RAG)

Vox Core is a **backend/orchestration project**, not an end-user product.

---

## 🚨 Read This First

**Do not start coding by reading this file alone.**

The authoritative entry point for understanding the project is:

👉 **`docs/PROJECT_OVERVIEW.md`**

That document:
- explains the project goals and scope
- defines non-negotiable security rules
- provides the documentation map and dependencies
- is required reading for humans *and* AI assistants

---

## Documentation Index

All system behavior is defined in the `docs/` directory.

Recommended reading order:

1. **PROJECT_OVERVIEW.md** – Project vision, scope, and document map  
2. **AI_COLLABORATION.md** – Rules for AI-assisted development  
3. **SECURITY_INVARIANTS.md** – Rules that must never be violated  
4. **ARCHITECTURE.md** – Components and data flow  
5. **DEPLOYMENT.md** – MiniPC-focused deployment guidelines  
6. **STATE_MACHINES.md** – Identity, conversation, command FSMs  
7. **DATA_MODELS.md** – Profiles, conversations, permissions  
8. **INTENTS.md** – Intent definitions and categories  
9. **INTENTS_COMMAND_MAPPING.md** – Which intents may execute commands  
10. **COMMANDS.md** – Command authorization and confirmation  
11. **IDENTITY_FUSION.md** – Voice + vision identity fusion  
12. **VISION.md** – Image-based perception  
13. **RAG.md** – Retrieval-Augmented Generation strategy  

---

## Key Principles (Summary)

- Identity is probabilistic and stateful
- Privacy is enforced by the core, not the LLM
- Commands require authorization and sometimes confirmation
- Delegation is explicit and revocable
- Ambiguity always resolves to safety
- Documentation is authoritative

---

## Project Structure (High Level)

```text
/
├── docs/
│   ├── PROJECT_OVERVIEW.md
│   ├── AI_COLLABORATION.md
│   ├── SECURITY_INVARIANTS.md
│   ├── ARCHITECTURE.md
│   ├── DEPLOYMENT.md
│   ├── STATE_MACHINES.md
│   ├── DATA_MODELS.md
│   ├── INTENTS.md
│   ├── INTENTS_COMMAND_MAPPING.md
│   ├── COMMANDS.md
│   ├── IDENTITY_FUSION.md
│   ├── VISION.md
│   └── RAG.md
├── server/
├── satellite/
├── TODO.md
└── README.md
```

---

## Contributing

Before contributing:
1. Read `docs/PROJECT_OVERVIEW.md`
2. Read `docs/AI_COLLABORATION.md`
3. Understand `docs/SECURITY_INVARIANTS.md`

If you are using an AI assistant:
- ensure it has access to the relevant docs
- ask for missing documents instead of guessing
- prefer small, incremental changes

---

## Status

Vox Core is under active design and early implementation.
Interfaces may evolve, but **security invariants must not be broken**.

---

## License

TBD
