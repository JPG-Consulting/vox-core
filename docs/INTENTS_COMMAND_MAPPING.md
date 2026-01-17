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
---

## No Fallback Execution

- The system MUST NOT execute commands based on semantic similarity,
  partial matches, or inferred “closest” intents.
- If an intent is not explicitly listed in this document,
  no command execution may occur.


> **Not every intent results in a command.**  
> Only intents explicitly mapped here MAY trigger command execution.

If an intent is not listed here, it MUST NOT execute a command.

---

## Intent Categories
---

## Identity and Preference Safety

- Intents related to identity, addressing, or personal preferences
  MUST NOT trigger command execution.
- Such intents MAY influence state machines but MUST NOT map to commands,
  even indirectly.
 and Command Eligibility

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

| Intent            | Command            | Group            | Required Permission        | Confirmation | Slots |
|-------------------|--------------------|------------------|---------------------------|--------------|-------|
| turn_on_light     | turn_on_light      | home_automation   | home_automation.execute   | No           | location |
| turn_off_light    | turn_off_light     | home_automation   | home_automation.execute   | No           | location |

---

### Door / Access Commands

| Intent            | Command            | Group            | Required Permission        | Confirmation | Slots |
|-------------------|--------------------|------------------|---------------------------|--------------|-------|
| close_door        | close_door         | home_automation   | home_automation.execute   | Yes          | door |
| open_garage       | open_garage        | home_automation   | home_automation.execute   | Yes          | garage |

---

### System / Data Commands

| Intent            | Command            | Group            | Required Permission        | Confirmation | Slots |
|-------------------|--------------------|------------------|---------------------------|--------------|-------|
| delete_all_data   | delete_all_data    | system           | system.execute            | Yes          | — |

---

## Group Authorization
---

## Age Enforcement

- Intent-to-command mapping MUST respect the requester’s age classification.
- Mappings that resolve to HIGH-risk commands MUST be rejected if the requester
  is classified as CHILD or TEEN.
- Age classification MUST be re-evaluated at confirmation time.

### Delegation State Binding

- Intent-to-command execution MUST re-check delegation state at execution time.
- If delegation has been revoked, the command MUST NOT execute,
  even if the intent mapping is otherwise valid.


- Each command mapping MUST specify a command group.
- Authorization MUST check the requester has the group permission (e.g., `home_automation.execute`).
- Delegation MAY grant access to some groups but not others.

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


---

## Confirmation Authority Binding

- When confirmation is required, it MUST satisfy all rules defined in `COMMANDS.md`,
  including:
  - same confirmed identity
  - same age classification
  - same conversation context
- Mapping tables MUST NOT override confirmation authority rules.


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
- 2026-01-16: Added command groups and group-based permissions to mappings.
