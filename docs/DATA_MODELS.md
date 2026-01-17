# DATA_MODELS.md

## Purpose

This document defines the **authoritative data models** used by the system.

These models:
- are implementation-agnostic
- define structure, scope, and intent
- enforce separation between private and shared data

All models MUST respect `SECURITY_INVARIANTS.md`.

---

## General Principles

- All models MUST include provenance (source of data).
- All sensitive fields MUST include visibility rules.
- Partial / unknown data MUST be representable.
- No implicit data merging is allowed.

---

## User Profile

Represents long-lived, private information about a user.

json
{
  "user_id": "user_1",
  "display_name": "Juan",
  "role": "OWNER",
  "locale": "es-ES",
  "profile": {
    "birthdate": {
      "day": 7,
      "month": 12,
      "year": 1977,
      "visibility": "private",
      "confidence": "high",
      "source": "voice_statement"
    }
  },
  "language_preferences": {
    "locale": "es-ES",
    "addressing": {
      "grammatical_gender": "masculine | feminine | neutral | unknown",
      "confidence": "high | medium | low",
      "source": "explicit | inferred | default"
    }
  },
  "voiceprints": [
    {
      "embedding_id": "vp_001",
      "confidence": "high",
      "created_at": "2026-01-10"
    }
  ]
}


### Notes

- `visibility` may be: private | shared | public
- Partial dates MUST allow missing year/month/day
- Profile data MUST NOT be exposed by default
- Grammatical gender refers to linguistic form, not biological sex

---

## Conversation

Represents a single interaction session.

json
{
  "conversation_id": "abc123",
  "mode": "SHARED_VERIFIED",
  "participants": ["user_1", "user_2"],
  "started_at": "2026-01-16T10:00:00Z",
  "memory": []
}


---

## Conversation Memory Entry

Stores facts or statements made during a conversation.

json
{
  "speaker": "juan",
  "type": "fact",
  "key": "birthdate",
  "value": { "day": 7, "month": 12 },
  "confidence": "medium",
  "timestamp": "2026-01-16T10:05:00Z"
}


### Notes

- Conversation memory is scoped to the conversation.
- It MAY be used to answer questions from participants.
- It MUST NOT be merged automatically into user profiles.

---

## Roles

Defines baseline user capabilities.

json
{
  "OWNER": ["all_permissions"],
  "ADMIN": ["manage_users", "delegate_permissions"],
  "OPERATOR": [],
  "GUEST": []
}


---

## Command Groups

Commands are organized into **command groups** for coarse-grained authorization.

Examples:
- home_automation
- robot_control
- security
- media
- system

Groups are the primary unit of permission assignment.

---

## Group-Based Permissions

Permissions are granted at the **command group** level.

json
{
  "group": "home_automation",
  "allowed_actions": ["execute"]
}


Rules:
- Granting a group permission allows execution of all commands in that group
- Group permissions do NOT imply permission for other groups
- Fine-grained checks (confirmation, context) still apply

---

## Delegation

Temporary permission grant from one user to another.

json
{
  "delegation_id": "del_001",
  "granted_by": "juan",
  "granted_to": "miguel",
  "groups": ["home_automation"],
  "scope": "conversation",
  "expires_at": "conversation_end",
  "active": true
}


Rules:
- Delegation is group-scoped
- Delegated users MUST NOT re-delegate
- Delegation expiration MUST be enforced

---

## Consent

Explicit authorization to disclose private data.

json
{
  "consent_id": "cons_001",
  "owner": "juan",
  "granted_to": ["miguel"],
  "data_keys": ["birthdate"],
  "scope": "conversation",
  "expires_at": "conversation_end",
  "active": true
}


---

## Pending Command

Represents a command awaiting confirmation.

json
{
  "command_id": "cmd_001",
  "intent": "close_door",
  "requested_by": "juan",
  "group": "home_automation",
  "requires_confirmation": true,
  "status": "pending_confirmation",
  "created_at": "2026-01-16T10:06:00Z"
}


---

## Identity Signal

Represents a single identity inference input.

json
{
  "source": "voice",
  "user_id": "user_1",
  "confidence": 0.87
}


---

## Field Mutability and Authority Metadata

Certain profile fields are subject to immutability or restricted update rules.

Models MAY include metadata to make these constraints explicit to implementations.

Example:

json
{
  "field": "profile.birthdate",
  "mutable_by": ["owner", "authorized_adult"],
  "locked_when": ["age_group == child", "age_group == teen"],
  "requires_audit": true
}


Rules:
- Fields marked as owner-only MUST NOT be modified by third parties.
- Fields locked by age MUST reject direct modification attempts.
- Restricted fields SHOULD require an explicit update flow, not direct mutation.


---

## Pending Profile Update

Represents a proposed change to profile data that requires validation or confirmation.

json
{
  "update_id": "upd_001",
  "target_user_id": "user_2",
  "field": "addressing.preferred_name",
  "proposed_value": "Mike",
  "proposed_by": "miguel",
  "reason": "explicit_user_request",
  "status": "pending | approved | rejected",
  "created_at": "2026-01-16T10:10:00Z"
}


Rules:
- Profile data MUST NOT be updated directly from conversation memory.
- Updates MUST transition through a pending state when confirmation or authority checks are required.


### Birthdate Authority Rules (Model-Level)

- Birthdate fields SHOULD be treated as restricted fields.
- If `age_group` is `child` or `teen`, direct mutation MUST be disallowed.
- Birthdate updates SHOULD be represented as pending profile updates.
- Implementations MUST record audit metadata for birthdate changes.

Addressing preferences are owner-controlled fields.

- Only the profile owner may confirm or change these values.
- Third-party statements or inference MUST NOT directly modify addressing fields.

---

## Summary

- User profiles are private and long-lived
- Language and addressing preferences are first-class
- Permissions are group-based
- Delegation is explicit and revocable
- Pending objects are first-class
- Provenance and visibility are mandatory

---

## Change Log

- 2026-01-16: Initial version.
- 2026-01-16: Added language & addressing preferences.
- 2026-01-16: Added group-based command permissions.
