# AI_COLLABORATION.md

## Purpose

This document defines how AI assistants (ChatGPT, Codex, GitHub Copilot, etc.) should collaborate on this repository.

Goals:
- Make AI contributions consistent, safe, and easy to review.
- Prevent accidental violations of privacy/security rules.
- Ensure documentation and code evolve together without contradictions.

Non-goals:
- This is not an implementation guide for a specific model/provider.
- This is not a product requirements document (PRD).
- This does not replace code review or testing.

---

## Documentation Safety and Change Discipline

All AI-assisted contributions to this repository MUST comply with
`DOCUMENTATION_POLICY.md`.

In particular:

- Authoritative documentation MUST NOT be rewritten from scratch.
- Changes to authoritative documents MUST be additive by default.
- Existing text MUST NOT be removed, shortened, or paraphrased unless the
  change is explicitly intended and documented.
- If an AI-generated change risks ambiguity, loss of constraints, or silent
  deletion of content, the change MUST be rejected.

AI assistants MUST:
- propose documentation changes as insertions, not replacements
- specify exact insertion points
- avoid reconstructing documents from summaries or memory

Violations of documentation safety rules are considered blocking issues.

---

## How to Use This Repo as an AI Assistant

### MUST Read First (in this order)

1. `docs/PROJECT_OVERVIEW.md`  
   The index and navigation map of the project.

2. `docs/SECURITY_INVARIANTS.md`  
   Hard rules that MUST NOT be violated. If any other document conflicts with it, this document wins.

3. `docs/ARCHITECTURE.md`  
   Components, boundaries, and high-level data flow.

4. `docs/STATE_MACHINES.md`  
   State machines for identity, conversation modes, commands, confirmations, and consent.

5. `docs/DATA_MODELS.md`  
   Schemas and data structures (profiles, conversations, permissions, memory scopes).

6. `docs/COMMANDS.md`  
   Command model, permissions, delegation, and confirmation-required commands.

If you do not have access to one of these documents, you MUST ask for it rather than guessing.

---

## Project Vocabulary (Use These Terms Consistently)

When writing code, docs, or discussing behavior, use this vocabulary:

- **Satellite**: ESP32-S3 / Raspberry Pi Zero 2W (client devices).
- **Server**: central backend that can provide STT/LLM/TTS and orchestration.
- **Orchestrator / Core**: main server logic; coordinates providers and enforces policies.
- **Provider**: pluggable implementation for STT, TTS, LLM, Vision, etc.
- **Identity**: who is speaking (or present) — never a simple boolean.
- **Conversation Mode**: social context (private/shared/unknown) that affects privacy.
- **Consent**: explicit authorization to disclose private data; scoped and revocable.
- **Role**: user capability baseline (OWNER/ADMIN/OPERATOR/GUEST).
- **Delegation**: temporary grant of specific permissions from one user to another.
- **Command**: an action request (e.g., home automation). Some commands require confirmation.

---

## Collaboration Rules for AI

### 1) Security and Privacy First (Fail Closed)

If any of these are uncertain, default to the safest behavior:
- Identity attribution
- Consent status
- Conversation participants/mode
- Authorization and delegation
- Command risk classification
- Data source boundaries (conversation memory vs user profile)

If uncertain:
- MUST NOT execute risky actions
- MUST NOT disclose private data
- SHOULD ask a clarifying question or request the missing document

### 2) The Core Enforces Policies, Not the LLM

You MUST NOT rely on “prompt rules” alone for:
- Privacy filtering
- Consent enforcement
- Authorization checks
- Delegation rules
- Confirmation gating for risky commands

These MUST be enforced by the server/core (policy engine / guards), with the LLM only generating text.

### 3) Data Sources Must Stay Separate

This repo distinguishes at least these memory sources:
- **Conversation Memory**: shared among participants of that conversation only.
- **User Profile**: private to the user unless explicitly shared or consented.

AI-generated changes MUST NOT introduce automatic cross-leaks between sources.

Example:
- If Miguel asks “When did Juan say his birthday is?” you may answer from conversation memory,
  but MUST NOT pull from Juan’s private profile unless consent or ownership rules allow it.

### 4) Confirmations Are Two-Phase and Explicit

Any sensitive change MUST use two-phase confirmation:
- granting permissions/delegations
- executing risky commands (e.g., “Close the door”)
- disclosing private data in shared settings (conservative policy)

Confirmations MUST be:
- explicit
- attributable to the correct user
- within the correct conversation context
- time-bounded (timeouts)

### 5) Identity is Sticky (Conservative)

Identity should not flip automatically during active conversation unless there is strong evidence.
When multiple people are present, privacy rules become stricter unless consent/ownership applies.

### 6) Prefer Incremental, Reviewable Changes

AI contributions MUST be:
- small and testable
- broken into phases (as described in project docs)
- accompanied by documentation updates when behavior changes

---

## When to Request Missing Context

If asked to implement or change behavior and you do not have:
- `SECURITY_INVARIANTS.md`
- `STATE_MACHINES.md`
- `DATA_MODELS.md`

You MUST request the missing document(s) rather than making assumptions.

---

## Documentation Update Policy

If an AI changes code behavior in a way that affects any of these:
- privacy
- identity resolution
- consent
- roles/delegation
- command execution / confirmations
- memory storage boundaries

Then the AI MUST update the relevant docs in the same change:
- update the owning document
- update `docs/PROJECT_OVERVIEW.md` if the doc map changes
- add an entry to the document’s change log section (if present)

---

## Recommended Output Format for AI Contributions

When proposing changes, the AI SHOULD structure responses as:

1. **Summary of intended change**
2. **Files to modify / add**
3. **Behavioral rules impacted** (with references to docs)
4. **Edge cases**
5. **Minimal test plan** (unit + integration)

---

## Test Expectations (High Level)

AI-generated code SHOULD include tests when feasible:
- unit tests for policy checks and state transitions
- integration tests for conversation flows (identity, mode changes, confirmations)

If tests cannot be added immediately, the AI MUST provide a written test plan.

---

## Change Log

- 2026-01-16: Initial version.
