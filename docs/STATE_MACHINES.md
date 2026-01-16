# STATE_MACHINES.md

## Purpose

This document defines the **explicit state machines (FSMs)** that govern system behavior.
All non-trivial logic MUST be driven by these FSMs.

This document depends on:
- PROJECT_OVERVIEW.md
- SECURITY_INVARIANTS.md
- ARCHITECTURE.md

If any behavior contradicts this document, it MUST be corrected.

---

## Overview of State Machines

The system defines the following core FSMs:

1. Identity Resolution FSM
2. Conversation Mode FSM
3. Command Execution FSM
4. Permission Delegation FSM
5. Consent FSM

Each FSM is independent but may influence others through well-defined events.

---

## 1) Identity Resolution FSM

### States

- UNKNOWN
- PROBABLE
- CONFIRMED
- CONFIRMED_ACTIVE
- AMBIGUOUS
- REJECTED

### Events

- voice_signal(confidence)
- face_signal(confidence)
- satellite_identity(confidence)
- explicit_identity_claim(name)
- validation_success
- validation_failure
- timeout
- strong_conflict

### Transitions

```text
UNKNOWN
  ├─ signal_low → UNKNOWN
  ├─ signal_medium → PROBABLE
  └─ explicit_claim → PROBABLE

PROBABLE
  ├─ validation_success → CONFIRMED
  ├─ validation_failure → REJECTED
  ├─ strong_conflict → AMBIGUOUS
  └─ timeout → UNKNOWN

CONFIRMED
  └─ speaking_turn → CONFIRMED_ACTIVE

CONFIRMED_ACTIVE
  ├─ silence_timeout → CONFIRMED
  ├─ strong_conflict → AMBIGUOUS
  └─ end_conversation → UNKNOWN

AMBIGUOUS
  ├─ clarification_success → CONFIRMED
  └─ timeout → UNKNOWN

REJECTED
  └─ timeout → UNKNOWN
```

### Notes

- Identity lock applies in CONFIRMED_ACTIVE.
- Identity MUST NOT downgrade automatically without strong evidence.

---

## 2) Conversation Mode FSM

### States

- PRIVATE
- SHARED_VERIFIED
- SHARED_UNVERIFIED

### Events

- participant_detected(identity_known)
- participant_left
- explicit_statement("someone arrived")
- explicit_statement("we are alone")
- identity_confirmed
- identity_unknown

### Transitions

```text
PRIVATE
  ├─ participant_detected(known) → SHARED_VERIFIED
  ├─ participant_detected(unknown) → SHARED_UNVERIFIED

SHARED_UNVERIFIED
  ├─ identity_confirmed → SHARED_VERIFIED
  └─ participant_left → PRIVATE

SHARED_VERIFIED
  ├─ participant_left(last) → PRIVATE
  └─ identity_unknown → SHARED_UNVERIFIED
```

### Notes

- Mode changes affect privacy rules immediately.
- Mode changes MUST be explicit or strongly detected.

---

## 3) Command Execution FSM

### States

- IDLE
- PENDING_AUTHORIZATION
- PENDING_CONFIRMATION
- EXECUTED
- DENIED
- CANCELLED

### Events

- command_detected
- authorization_granted
- authorization_denied
- confirmation_required
- confirmation_received
- confirmation_timeout
- execution_success
- execution_failure

### Transitions

```text
IDLE
  └─ command_detected → PENDING_AUTHORIZATION

PENDING_AUTHORIZATION
  ├─ authorization_denied → DENIED
  ├─ authorization_granted & immediate → EXECUTED
  └─ authorization_granted & requires_confirmation → PENDING_CONFIRMATION

PENDING_CONFIRMATION
  ├─ confirmation_received → EXECUTED
  ├─ confirmation_timeout → CANCELLED
  └─ context_changed → CANCELLED

EXECUTED
  └─ done → IDLE

DENIED
  └─ done → IDLE

CANCELLED
  └─ done → IDLE
```

---

## 4) Permission Delegation FSM

### States

- NONE
- PENDING_CONFIRMATION
- ACTIVE
- REVOKED
- EXPIRED

### Events

- delegation_intent_detected
- owner_confirmation
- owner_denial
- revoke_command
- timeout
- conversation_end

### Transitions

```text
NONE
  └─ delegation_intent_detected → PENDING_CONFIRMATION

PENDING_CONFIRMATION
  ├─ owner_confirmation → ACTIVE
  ├─ owner_denial → NONE
  └─ timeout → NONE

ACTIVE
  ├─ revoke_command → REVOKED
  ├─ conversation_end → EXPIRED

REVOKED
  └─ done → NONE

EXPIRED
  └─ done → NONE
```

### Notes

- Delegation never changes the underlying role.
- Delegation is always scoped and temporary.

---

## 5) Consent FSM

### States

- NONE
- PENDING
- GRANTED
- EXPIRED
- REVOKED

### Events

- consent_request
- explicit_grant
- explicit_denial
- timeout
- conversation_end

### Transitions

```text
NONE
  └─ consent_request → PENDING

PENDING
  ├─ explicit_grant → GRANTED
  ├─ explicit_denial → NONE
  └─ timeout → NONE

GRANTED
  ├─ conversation_end → EXPIRED
  └─ revoke → REVOKED

REVOKED
  └─ done → NONE

EXPIRED
  └─ done → NONE
```

---

## Cross-FSM Rules

- Identity FSM influences all others.
- Conversation Mode FSM affects Privacy Guard decisions.
- Command FSM MUST consult Authorization and Consent FSMs.
- Delegation FSM never bypasses Confirmation FSM.

---

## Summary

- All behavior is state-driven.
- No implicit transitions.
- No side effects without explicit events.
- Ambiguity always resolves to safety.

---

## Change Log

- 2026-01-16: Initial version.
