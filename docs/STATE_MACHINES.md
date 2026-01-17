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
2. Addressing Resolution Sub-State
3. Age Context Derivation FSM
4. Conversation Mode FSM
5. Command Execution FSM
6. Permission Delegation FSM
7. Consent FSM
8. System Event FSM (Date-aware events)

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


### Identity Downgrade Guards

- CONFIRMED and CONFIRMED_ACTIVE states MUST NOT downgrade automatically due to
  timeouts or minor confidence decay.
- Downgrades from confirmed states require a strong_conflict event.


---


---

## 1.1) Addressing Resolution Sub-State

Addressing resolution determines **how the system speaks to the user** in gendered languages and how it addresses them by name.

This sub-state operates alongside identity resolution and SHOULD be resolved before:
- confirmations
- sensitive questions
- long-form responses

### States

- UNKNOWN
- INFERRED
- CONFIRMED

### Events

- explicit_preference(name_or_gender)
- inferred_signal(confidence)
- conflicting_signal

### Transitions (Conceptual)

```text
UNKNOWN
  ├─ inferred_signal(high) → INFERRED
  └─ explicit_preference → CONFIRMED

INFERRED
  ├─ explicit_preference → CONFIRMED
  └─ conflicting_signal → UNKNOWN

CONFIRMED
  └─ (sticky) remains CONFIRMED unless user changes preference
```

### Notes

- `CONFIRMED` addressing MUST override inferred addressing.
- If UNKNOWN, the system SHOULD ask once and store the answer.
- Addressing preferences MUST NOT be exposed to other users.


### Addressing Resolution Guards

- The system MUST NOT repeatedly ask for addressing preferences once a direct
  clarification question has been asked.
- If addressing preferences remain unresolved, the system MUST fall back to a
  neutral form of address.
- Addressing resolution MUST NOT block other state transitions.



---

## 1.2) Age Context Derivation FSM

Age context determines whether the system should use minor-safe behavior.
Age is derived from birthdate when available.

### States

- UNKNOWN
- CHILD
- TEEN
- ADULT

### Events

- birthdate_known
- date_tick
- birthdate_updated
- confidence_drop

### Notes

- If age is UNKNOWN, the system MUST behave as CHILD-safe.
- Age affects content policy, tone, and command restrictions.


### Age Mutation Guards

- Attempts to modify birthdate or age-related profile fields MUST trigger the
  Pending Profile Update FSM.
- If the current age group is CHILD or TEEN, self-initiated age or birthdate
  modifications MUST be rejected during validation.
- Age group transitions MUST NOT occur solely due to profile edit attempts.


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


---

## Pending Profile Update FSM

This state machine governs all modifications to user profile data that require
validation, authority checks, or explicit confirmation.

Direct mutation of profile data is forbidden outside this FSM.

### States

- NONE
- PENDING_VALIDATION
- PENDING_CONFIRMATION
- APPROVED
- REJECTED
- CANCELLED

### Events

- profile_update_requested
- authority_verified
- authority_denied
- owner_confirmation_received
- owner_confirmation_denied
- timeout
- context_changed

### Transitions

```text
NONE
  └─ profile_update_requested → PENDING_VALIDATION

PENDING_VALIDATION
  ├─ authority_verified & confirmation_required → PENDING_CONFIRMATION
  ├─ authority_verified & no_confirmation → APPROVED
  └─ authority_denied → REJECTED

PENDING_CONFIRMATION
  ├─ owner_confirmation_received → APPROVED
  ├─ owner_confirmation_denied → REJECTED
  └─ timeout → CANCELLED

APPROVED
  └─ applied → NONE

REJECTED
  └─ done → NONE

CANCELLED
  └─ done → NONE
```

### Invariants

- Profile data MUST NOT be modified outside the APPROVED state.
- Rejected or cancelled updates MUST NOT partially apply.
- Only one pending profile update per user MAY exist at a time.


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


---

## 6) System Event FSM (Date-aware Events)

The system may emit proactive events based on trusted time/date tools.
These events MUST NOT bypass privacy, consent, or authorization rules.

### Birthday Greeting Event

Conditions:
- identity == CONFIRMED (or CONFIRMED_ACTIVE)
- current date is trusted
- birthdate (month/day) is known with sufficient confidence
- greeting not yet emitted for this user on this date

Action:
- emit `birthday_greeting(user_id)`

Notes:
- Greet once per day.
- Do not speak full birthdate unless allowed.
- In SHARED modes, greet only if it does not reveal sensitive info.


### Birthday Event Guards

- Birthday greetings MUST NOT be emitted if identity is ambiguous at the time
  of emission.
- In SHARED modes, birthday greetings MUST NOT be emitted for users classified
  as CHILD or TEEN.
- Identity state and conversation mode MUST be re-evaluated immediately before
  emitting the event.


## Cross-FSM Rules

- Identity FSM influences all others.
- Addressing Resolution influences spoken output (name and grammatical gender).
- Age Context influences content restrictions and command gating.
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
- 2026-01-16: Added addressing resolution, age context derivation, and date-aware system events.
