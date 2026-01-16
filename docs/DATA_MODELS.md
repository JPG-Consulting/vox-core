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

```json
{
  "user_id": "juan",
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
  "voiceprints": [
    {
      "embedding_id": "vp_001",
      "confidence": "high",
      "created_at": "2026-01-10"
    }
  ]
}
```

### Notes

- `visibility` may be: private | shared | public
- Partial dates MUST allow missing year/month/day
- Profile data MUST NOT be exposed by default

---

## Conversation

Represents a single interaction session.

```json
{
  "conversation_id": "abc123",
  "mode": "SHARED_VERIFIED",
  "participants": ["juan", "miguel"],
  "started_at": "2026-01-16T10:00:00Z",
  "memory": []
}
```

---

## Conversation Memory Entry

Stores facts or statements made during a conversation.

```json
{
  "speaker": "juan",
  "type": "fact",
  "key": "birthdate",
  "value": { "day": 7, "month": 12 },
  "confidence": "medium",
  "timestamp": "2026-01-16T10:05:00Z"
}
```

### Notes

- Conversation memory is scoped to the conversation.
- It MAY be used to answer questions from participants.
- It MUST NOT be merged automatically into user profiles.

---

## Roles

Defines baseline user capabilities.

```json
{
  "OWNER": ["all_permissions"],
  "ADMIN": ["execute_commands", "manage_users"],
  "OPERATOR": ["execute_commands"],
  "GUEST": []
}
```

---

## Permissions

Permissions are fine-grained capabilities.

Examples:
- execute_commands
- manage_users
- control_lights
- control_doors

Permissions are evaluated dynamically.

---

## Delegation

Temporary permission grant from one user to another.

```json
{
  "delegation_id": "del_001",
  "granted_by": "juan",
  "granted_to": "miguel",
  "permissions": ["execute_commands"],
  "scope": "conversation",
  "expires_at": "conversation_end",
  "active": true
}
```

---

## Consent

Explicit authorization to disclose private data.

```json
{
  "consent_id": "cons_001",
  "owner": "juan",
  "granted_to": ["miguel"],
  "data_keys": ["birthdate"],
  "scope": "conversation",
  "expires_at": "conversation_end",
  "active": true
}
```

---

## Pending Command

Represents a command awaiting confirmation.

```json
{
  "command_id": "cmd_001",
  "intent": "close_door",
  "requested_by": "juan",
  "requires_confirmation": true,
  "status": "pending_confirmation",
  "created_at": "2026-01-16T10:06:00Z"
}
```

---

## Identity Signal

Represents a single identity inference input.

```json
{
  "source": "voice",
  "user_id": "juan",
  "confidence": 0.87
}
```

---

## Summary

- User profiles are private and long-lived
- Conversation memory is shared and temporary
- Permissions and delegations are explicit
- Pending objects are first-class
- Provenance and visibility are mandatory

---

## Change Log

- 2026-01-16: Initial version.
