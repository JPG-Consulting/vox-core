# SECURITY_INVARIANTS.md

## Purpose

This document defines the **non-negotiable security and privacy rules** of the system.

These rules:
- MUST ALWAYS be enforced by the core/server.
- MUST NOT be overridden by prompts, models, or UI logic.
- Take precedence over all other documentation.

If any other document, code, or AI-generated change conflicts with this document,
**this document wins**.

---

## Fundamental Principle

> **When in doubt, do nothing.**  
> Fail closed, never fail open.

If identity, consent, authorization, or intent is unclear:
- Do NOT execute commands
- Do NOT disclose private data
- Ask for clarification or confirmation

---

## Identity Invariants

### INV-IDENTITY-1: Identity Is Probabilistic

- Identity MUST be treated as probabilistic and stateful.
- Identity MUST NOT be assumed from a single signal.
- Voice, image, satellite-provided identity, and user statements are signals, not facts.

---

### INV-IDENTITY-2: Identity Is Sticky During Active Conversation

- Once an identity is CONFIRMED and active, it MUST remain locked
  unless strong evidence indicates otherwise.
- Identity MUST NOT change automatically due to weak or ambiguous signals.

---

### INV-IDENTITY-3: No Implicit Identity Escalation

- The system MUST NOT escalate identity confidence automatically.
- Confirmation, verification, or explicit user action is required.

---

## Privacy Invariants

### INV-PRIVACY-1: Private Data Is Private by Default

- All user profile data is PRIVATE unless explicitly marked otherwise.
- Private data MUST NOT be disclosed to other users without consent.

---

### INV-PRIVACY-2: Data Sources MUST NOT Be Mixed

The system distinguishes at least:
- Conversation memory
- User profile memory

Rules:
- Conversation memory MAY be shared among participants of that conversation.
- User profile data MUST NOT be used to answer questions for other users
  unless explicit consent exists.

---

### INV-PRIVACY-3: Ownership Allows Self-Disclosure Only

- A user MAY access their own private data if:
  - their identity is CONFIRMED
  - the request context is safe (see conversation modes)
- In shared conversations, conservative policy applies:
  - private data disclosure requires explicit confirmation

---

### INV-PRIVACY-4: Consent Must Be Explicit

- Consent MUST be:
  - explicitly stated
  - attributable to the data owner
  - scoped (what data, who can receive it)
  - temporary (conversation or time-bounded)

- Consent MUST NOT be:
  - inferred
  - assumed
  - permanent by default

---

## Conversation Mode Invariants

### INV-CONV-1: Conversation Mode Controls Disclosure

Conversation mode determines what may be said, not who is speaking.

At minimum:
- PRIVATE
- SHARED_VERIFIED
- SHARED_UNVERIFIED

Rules:
- In SHARED modes, private data MUST NOT be disclosed
  unless consent or ownership rules allow it.

---

### INV-CONV-2: Mode Changes Are Explicit

- Conversation mode MUST NOT change implicitly.
- Mode changes require:
  - explicit user statement
  - or strong, verified detection (e.g., confirmed new participant)

---

## Command Authorization Invariants

### INV-CMD-1: Commands Require Authorization

- Every command MUST define required permissions.
- A command MUST NOT execute unless the requester is authorized.

Knowing someone ≠ being authorized.

---

### INV-CMD-2: Delegation Is Explicit and Limited

- Delegation MUST be:
  - granted explicitly
  - granted by a user with sufficient authority
  - scoped (permissions, duration, context)

- Delegation MUST NOT:
  - grant more permissions than the grantor has
  - allow re-delegation unless explicitly allowed

---

### INV-CMD-3: No Permission Without Confirmation

- Granting permissions MUST use a two-phase flow:
  1. detect intent
  2. explicit confirmation by the owner

Ambiguous phrases MUST NOT grant permissions.

---

## Command Confirmation Invariants

### INV-CMD-4: Risky Commands Require Confirmation

Commands classified as risky MUST:
- require explicit confirmation
- be executed only after confirmation
- expire if confirmation is not received in time

---

### INV-CMD-5: Confirmation Must Come From the Requester

- Confirmation MUST come from:
  - the same identity that requested the command
  - with identity still confirmed
- Confirmations from other users MUST be ignored.

---

## Multi-User Safety Invariants

### INV-MULTI-1: Presence Increases Restrictions

- When multiple users are present:
  - privacy rules become stricter
  - confirmations become mandatory for sensitive actions

---

### INV-MULTI-2: No Cross-User Leakage

- The system MUST NOT:
  - reveal one user’s private data to another
  - infer permissions based on social presence

---

## LLM Usage Invariants

### INV-LLM-1: LLM Is Not a Policy Engine

- The LLM MUST NOT:
  - decide authorization
  - decide consent
  - decide identity
  - bypass confirmations

All enforcement MUST happen in the core.

---

### INV-LLM-2: Context Filtering Is Mandatory

- The LLM MUST only receive:
  - data that is explicitly allowed
  - no hidden or excess context

If the LLM does not see it, it cannot leak it.

---

## Failure Handling Invariants

### INV-FAIL-1: Fail Closed

If any of the following fail:
- STT
- identity resolution
- permission checks
- confirmation logic

Then:
- commands MUST NOT execute
- private data MUST NOT be disclosed

---

### INV-FAIL-2: Errors Must Be User-Safe

On failure, the system SHOULD:
- apologize briefly
- ask to repeat or clarify
- reset pending states safely

---

## Summary (Authoritative)

1. Identity is never assumed
2. Privacy is default
3. Consent is explicit
4. Authorization is mandatory
5. Risky actions require confirmation
6. Ambiguity results in no action
7. The core enforces all rules
8. The LLM only generates text

---

## Change Log

- 2026-01-16: Initial version.
