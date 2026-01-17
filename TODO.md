# TODO.md

## Purpose

This document tracks **pending work, open questions, and planned improvements**
for the project.

It is intended to:
- keep the repository tidy
- provide a shared roadmap for humans and AI assistants
- avoid scattering TODOs across code and documentation

This file is **not a commitment or timeline**.
Items may change priority or be removed as the project evolves.

---

## Guiding Rules

- Items here are intentionally high-level.
- Detailed implementation tasks belong in issues or PRs.
- If a TODO affects documented behavior, the relevant docs MUST be updated when completed.
- Security and privacy TODOs have priority over features.

---

## Phase 0 – Project Foundations (Current)

### Documentation
- [x] Core documentation set defined
- [x] Security invariants formalized
- [x] State machines formalized
- [ ] Add addressing resolution (grammatical gender + preferred name)
- [x] Deployment constraints defined
- [ ] Review documentation for consistency after first code scaffolding
- [ ] Add diagrams (optional, non-authoritative)

### Repository Structure
- [ ] Define final directory layout for `server/` and `satellite/`
- [ ] Decide on configuration file format(s)
- [ ] Add `.editorconfig` and basic linting rules

---

## Phase 1 – Audio Output (TTS First)

### TTS Pipeline
- [ ] Define TTS provider interface
- [ ] Integrate initial TTS engine (e.g., Piper)
- [ ] Implement streaming TTS output
- [ ] Implement TTS cancellation (barge-in)
- [ ] Support multiple audio codecs (PCM, Opus, MP3)

### Performance
- [ ] Voice pre-warming at startup
- [ ] Measure TTFA (time to first audio)

---

## Phase 2 – Audio Input (STT)

### STT Pipeline
- [ ] Define STT provider interface
- [ ] Integrate initial STT engine
- [ ] Support streaming transcription
- [ ] Speaker embedding extraction (optional)
- [ ] Confidence propagation from STT

### Satellite Interaction
- [ ] Decide what STT runs on satellite vs server
- [ ] Define audio chunking protocol

---

## Phase 3 – LLM Integration

### LLM Core
- [ ] Define LLM provider interface
- [ ] Integrate local LLM (e.g., Ollama)
- [ ] Implement streaming token output
- [ ] Implement tool / function calling
- [ ] Implement cancellation on barge-in

### Context Management
- [ ] Context filtering per privacy rules
- [ ] Date-aware system events (birthday greeting)
- [ ] Conversation memory injection
- [ ] System-time and date tools

---

## Phase 4 – Commands & Automation

### Command System
- [ ] Implement group-based command permissions
- [ ] Implement command execution pipeline
- [ ] Implement authorization manager
- [ ] Implement confirmation manager
- [ ] Implement delegation lifecycle
- [ ] Implement audit logging for commands

### Safety
- [ ] Add per-command risk metadata
- [ ] Add command execution timeouts

---

## Phase 5 – Identity & Multi-User

### Identity
- [ ] Implement age context derivation and minor-safe defaults
- [ ] Implement identity FSM
- [ ] Implement speaker recognition
- [ ] Implement identity lock logic
- [ ] Implement clarification flows

### Multi-User
- [ ] Conversation mode transitions
- [ ] Privacy restrictions in shared mode
- [ ] Multiple concurrent speakers handling

---

## Phase 6 – Vision (Optional / Future)

### Vision Pipeline
- [ ] Define vision provider interface
- [ ] Implement face detection
- [ ] Implement face identification
- [ ] Satellite-provided vision identity support
- [ ] Vision + speaker fusion logic

---

## Phase 7 – RAG & External Knowledge

### RAG
- [ ] Define knowledge store schema
- [ ] Implement retrieval pipeline
- [ ] Integrate first external LLM fallback
- [ ] Implement consent-based storage
- [ ] Add provenance and confidence tracking

---

## Cross-Cutting Concerns

### Security & Privacy
- [ ] Implement third-party personal-fact provenance and visibility rules
- [ ] Pen-test privacy edge cases
- [ ] Verify fail-closed behavior
- [ ] Review all external API calls

### Testing
- [ ] Unit tests for FSMs
- [ ] Integration tests for voice flows
- [ ] Multi-user scenario tests

### Observability
- [ ] Structured logging
- [ ] Latency metrics
- [ ] Minimal audit log review tools

---

## Open Questions

- [ ] Final list of supported audio codecs
- [ ] Persistence strategy for conversation memory
- [ ] Long-term identity verification strategy
- [ ] Per-user vs global RAG knowledge scope

---

## Notes for AI Assistants

If you are an AI assistant:
- Treat this file as a roadmap, not strict instructions
- Do not implement items here without checking relevant docs
- Prefer small, incremental changes
- Ask for clarification if priorities are unclear

---

## Change Log

- 2026-01-16: Initial version.
- 2026-01-16: Added tasks for addressing, age-aware behavior, command grouping, and third-party fact provenance.
