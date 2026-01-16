# COMMANDS.md

## Purpose

This document defines the **command system** of the project:
- command taxonomy
- permission requirements
- immediate vs confirmation-required commands
- execution and denial behavior
- interaction with roles, delegation, and consent

All command behavior MUST respect `SECURITY_INVARIANTS.md`.

---

## Command Definition

A command is a structured intent that may trigger an action in the real world or system.

Each command MUST define:
- intent identifier
- required permissions
- risk level
- confirmation policy

Example:

```json
{
  "intent": "turn_on_light",
  "risk_level": "low",
  "command_type": "IMMEDIATE",
  "required_permissions": ["control_lights"]
}
```

---

## Command Types

### 1) IMMEDIATE Commands

Characteristics:
- low risk
- reversible
- no physical safety impact

Examples:
- turn_on_light
- turn_off_light
- set_volume
- pause_music

Rules:
- MAY be executed immediately if authorized
- MUST NOT require confirmation

---

### 2) CONFIRM_REQUIRED Commands

Characteristics:
- physical or security risk
- irreversible or sensitive actions

Examples:
- close_door
- open_garage
- disable_alarm
- delete_all_data

Rules:
- MUST require explicit confirmation
- MUST create a pending command
- MUST expire if not confirmed

Example:

```json
{
  "intent": "close_door",
  "risk_level": "high",
  "command_type": "CONFIRM_REQUIRED",
  "required_permissions": ["control_doors"]
}
```

---

## Authorization Rules

A command may only execute if:
1. The speaker identity is CONFIRMED
2. The speaker has the required permissions via:
   - role
   - active delegation
3. Conversation mode allows command execution

Knowing a user does not grant permission.

---

## Delegation Interaction

Delegation:
- allows execution of specific commands
- does NOT change user role
- does NOT allow re-delegation

Delegated users:
- MAY execute allowed commands
- MUST NOT grant permissions to others

---

## Confirmation Flow (Two-Phase)

For CONFIRM_REQUIRED commands:

1. Command is detected
2. Authorization is checked
3. Pending command is created
4. System asks for confirmation
5. Confirmation is evaluated
6. Command is executed or cancelled

Confirmation rules:
- MUST come from the same user
- MUST be explicit
- MUST occur before timeout
- MUST match the pending command

---

## Denial Behavior

If a command is denied, the system SHOULD:
- explain briefly why
- not reveal sensitive details
- remain polite and neutral

Examples:
- “I don’t have permission to do that.”
- “I need confirmation before doing that.”
- “I can’t do that while others are present.”

---

## Conversation Mode Interaction

- PRIVATE:
  - commands allowed if authorized
- SHARED_VERIFIED:
  - commands allowed if authorized
  - risky commands require confirmation
- SHARED_UNVERIFIED:
  - commands SHOULD be restricted
  - risky commands SHOULD be denied or require clarification

---

## Command Execution Pipeline (Summary)

```text
speech
  → STT
    → intent detection
      → authorization check
        → confirmation check (if required)
          → execution
            → result
```

At any step, failure results in safe denial.

---

## Error Handling

If execution fails:
- command MUST NOT partially execute
- system SHOULD notify user safely
- no retry without new request

---

## Non-Goals

- Natural language command discovery
- Autonomous command chaining
- Implicit permission escalation

---

## Summary

- Commands are explicit, structured, and gated
- Authorization is mandatory
- Risky commands require confirmation
- Delegation is limited and temporary
- Ambiguity results in no action

---

## Change Log

- 2026-01-16: Initial version.
