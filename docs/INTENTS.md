# INTENTS.md

## Purpose

This document defines the **intent system** of the project.

An intent represents the **interpreted meaning** of a user utterance.
Not all intents result in commands; many are informational or conversational.

This document defines:
- intent categories
- intent schema
- relationship between intents and commands
- safety and ambiguity handling

INTENTS MUST be interpreted and validated by the Core, not trusted blindly from the LLM.

---

## Definition of an Intent

An intent is a structured representation derived from user input.

Minimal intent schema:

```json
{
  "intent": "turn_on_light",
  "category": "command",
  "confidence": 0.92,
  "slots": {
    "location": "living_room"
  }
}
```

Fields:
- `intent`: unique identifier
- `category`: intent type (see below)
- `confidence`: model confidence (0.0–1.0)
- `slots`: extracted parameters (may be partial or empty)

---

## Intent Categories

### 1) Conversational Intents

Used for dialogue, not actions.

Examples:
- greet
- ask_name
- ask_identity
- small_talk
- help

Rules:
- MUST NOT trigger commands
- MAY update conversation state
- MAY update conversation memory

Example:
```json
{
  "intent": "greet",
  "category": "conversation"
}
```

---

### 2) Informational Intents

Requests for information.

Examples:
- ask_time
- ask_date
- ask_weather
- ask_conversation_fact

Rules:
- MAY use tools (time, date, calculations)
- MUST respect privacy rules
- MUST NOT access private profiles of others without consent

Example:
```json
{
  "intent": "ask_time",
  "category": "information"
}
```

---

### 3) Identity Intents

Related to identity, presence, or participants.

Examples:
- identify_self
- identify_other
- new_participant_arrived
- participant_left

Rules:
- MUST feed Identity FSM
- MUST NOT directly change identity without validation

Example:
```json
{
  "intent": "identify_self",
  "category": "identity",
  "slots": { "name": "Juan" }
}
```

---

### 4) Permission / Control Intents

Related to permissions, delegation, and control.

Examples:
- grant_permission
- revoke_permission
- request_permission
- confirm_permission

Rules:
- MUST follow two-phase confirmation
- MUST be attributable to owner
- MUST NOT auto-execute

Example:
```json
{
  "intent": "grant_permission",
  "category": "permission",
  "slots": {
    "target_user": "miguel",
    "permission": "execute_commands"
  }
}
```

---

### 5) Command Intents

Intents that map directly to executable commands.

Examples:
- turn_on_light
- close_door
- open_garage

Rules:
- MUST map to a command definition in COMMANDS.md
- MUST pass authorization checks
- MAY require confirmation depending on command type

Example:
```json
{
  "intent": "close_door",
  "category": "command",
  "slots": {
    "door": "front"
  }
}
```

---

## Intent Confidence Handling

- Confidence MUST be considered by the Core.
- Low-confidence intents SHOULD trigger clarification.
- High-risk intents with low confidence MUST NOT execute.

Example clarification:
> “Did you mean to close the door?”

---

## Intent → Command Mapping

Not all intents become commands.

Mapping rules:
- category == command → candidate for execution
- other categories → handled conversationally

Mapping MUST:
- validate required slots
- validate permissions
- respect confirmation policy

---

## Ambiguity Handling

If:
- multiple intents are possible
- intent confidence is low
- slots are incomplete

Then:
- system MUST ask a clarification question
- system MUST NOT guess

---

## Non-Goals

- Autonomous intent chaining
- Implicit intent escalation
- Learning new intents without review

---

## Summary

- Intents represent meaning, not actions
- Commands are a subset of intents
- Confidence and category matter
- Ambiguity always resolves to clarification
- The Core validates everything

---

## Change Log

- 2026-01-16: Initial version.
