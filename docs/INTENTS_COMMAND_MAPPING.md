# INTENTS_COMMAND_MAPPING.md

## Purpose

This document defines the **authoritative mapping between INTENTS and COMMANDS**.

It answers:
- which intents can trigger commands
- which intents never trigger commands
- how intent slots map to command parameters
- when clarification or confirmation is required

This document depends on:
- INTENTS.md
- COMMANDS.md
- SECURITY_INVARIANTS.md

---

## Core Rule

> **Not every intent results in a command.**  
> Only intents explicitly mapped here MAY trigger command execution.

If an intent is not listed here, it MUST NOT execute a command.

---

## Intent Categories and Command Eligibility

| Intent Category      | Can Trigger Command | Notes |
|----------------------|--------------------|-------|
| conversation         | ❌ No              | Dialogue only |
| information          | ❌ No              | Uses tools / RAG |
| identity             | ❌ No              | Feeds identity FSM |
| permission           | ❌ No              | Two-phase authorization only |
| command              | ✅ Yes             | Subject to mapping below |

---

## Command Mapping Table

### Lighting Commands

| Intent            | Command            | Required Permission | Confirmation | Slots |
|-------------------|--------------------|---------------------|--------------|-------|
| turn_on_light     | turn_on_light      | control_lights      | No           | location |
| turn_off_light    | turn_off_light     | control_lights      | No           | location |

---

### Door / Access Commands

| Intent            | Command            | Required Permission | Confirmation | Slots |
|-------------------|--------------------|---------------------|--------------|-------|
| close_door        | close_door         | control_doors       | Yes          | door |
| open_garage       | open_garage        | control_doors       | Yes          | garage |

---

### System / Data Commands

| Intent            | Command            | Required Permission | Confirmation | Slots |
|-------------------|--------------------|---------------------|--------------|-------|
| delete_all_data   | delete_all_data    | manage_system       | Yes          | — |

---

## Slot Validation Rules

- Required slots MUST be present
- Missing or ambiguous slots MUST trigger clarification
- Slot values MUST be normalized before execution

Example clarification:
> “Which door do you want me to close?”

---

## Confidence Thresholds

Suggested defaults:
- intent confidence < 0.6 → clarification required
- high-risk commands with confidence < 0.8 → deny + clarify

Thresholds MUST be configurable.

---

## Multi-User and Mode Interaction

- In SHARED_UNVERIFIED mode:
  - high-risk commands SHOULD be denied
- In SHARED_VERIFIED mode:
  - confirmation is mandatory for risky commands
- In PRIVATE mode:
  - standard rules apply

---

## Non-Goals

- Automatic discovery of new commands
- Executing commands based on partial intent
- Cross-command chaining

---

## Summary

- This file is the single source of truth for intent→command mapping
- Anything not listed here cannot execute
- Confirmation and permission rules always apply
- Ambiguity resolves to clarification

---

## Change Log

- 2026-01-16: Initial version.
