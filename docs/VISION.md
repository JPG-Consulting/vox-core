# VISION.md

## Purpose

This document defines how the system handles **visual input** (images) for:
- detecting the presence of people
- identifying known users (face recognition)
- producing identity signals for the Core

Vision is an **optional capability** and MUST integrate safely with identity,
privacy, and conversation rules defined elsewhere.

This document depends on:
- PROJECT_OVERVIEW.md
- SECURITY_INVARIANTS.md
- ARCHITECTURE.md
- STATE_MACHINES.md
- DATA_MODELS.md

---

## Scope

In scope:
- receiving images from satellites
- detecting faces / people
- identifying known users
- producing identity signals

Out of scope (for now):
- emotion recognition
- gaze tracking
- continuous video streaming
- biometric-grade guarantees

---

## Vision Endpoints

Vision MUST be handled via a **separate endpoint** from audio/voice.

Recommended endpoints:

```text
POST /vision/analyze
```

---

## Request Schema

```json
{
  "conversation_id": "abc123",
  "image": "<bytes | base64 | multipart>",
  "satellite_identity": {
    "user_id": "juan",
    "confidence": 0.95,
    "source": "face_local"
  },
  "options": {
    "detect_faces": true,
    "identify_faces": true
  }
}
```

Notes:
- `satellite_identity` is OPTIONAL
- If provided with high confidence, server-side analysis MAY be skipped
- Images MUST NOT be stored by default

---

## Response Schema

```json
{
  "faces": [
    {
      "user_id": "juan",
      "confidence": 0.91
    }
  ]
}
```

Special cases:
- Unknown person:
```json
{ "faces": [ { "user_id": "unknown" } ] }
```

- No faces detected:
```json
{ "faces": [] }
```

---

## Vision Processing Flow

1. Image received
2. Validate request and conversation context
3. If satellite identity is present and trusted:
   - accept as provisional identity signal
4. Else:
   - detect faces
   - extract embeddings
   - compare against known profiles
5. Produce identity signals
6. Pass signals to Identity Resolver

The Vision provider NEVER decides identity on its own.

---

## Identity Signals from Vision

Vision produces identity signals in the same format as other sources:

```json
{
  "source": "vision",
  "user_id": "juan",
  "confidence": 0.88
}
```

Rules:
- Vision signals are probabilistic
- They may reinforce or conflict with voice signals
- The Core resolves conflicts conservatively

---

## Multi-Person Images

If multiple faces are detected:
- Each face produces a separate identity signal
- Conversation Mode MAY change:
  - PRIVATE → SHARED_VERIFIED or SHARED_UNVERIFIED

Privacy rules become stricter immediately.

---

## Privacy and Safety Rules

- Vision MUST NOT expose identities of people not present in the conversation
- Vision MUST NOT reveal profile data
- Images MUST NOT be persisted unless explicitly configured
- Only minimal identity information may be returned

---

## Interaction with Conversation Mode

- Detecting a new face MAY trigger:
  - participant_detected event
- Identity MUST be confirmed before becoming active
- Mode changes MUST respect SECURITY_INVARIANTS.md

---

## Satellite-Provided Vision Identity

Satellites MAY:
- perform local face recognition
- send identity signals directly

Rules:
- Satellite signals are treated as probabilistic
- High-confidence signals MAY skip server-side vision
- Core MAY still downgrade confidence if conflicts arise

---

## Failure Handling

If vision processing fails:
- No identity is assumed
- No privacy-sensitive actions occur
- System MAY ask for clarification

---

## Summary

- Vision is optional and additive
- Identity from vision is probabilistic
- Vision never bypasses privacy or authorization
- Satellite-provided identity is supported
- Ambiguity always resolves to safety

---

## Change Log

- 2026-01-16: Initial version.
