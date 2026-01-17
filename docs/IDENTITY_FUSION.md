# IDENTITY_FUSION.md

## Purpose

This document defines how the system **fuses identity signals** coming from:
- speaker recognition (voice)
- vision (face recognition)
- satellite-provided identity
- explicit user statements

The goal is to determine:
- current speaker identity
- participants present
- confidence levels
- when identity is considered locked

This document depends on:
- SECURITY_INVARIANTS.md
- STATE_MACHINES.md
- DATA_MODELS.md
- VISION.md

---

## Core Principle

> **Identity is inferred, not assumed.**  
> Multiple weak signals never outweigh a strong conflict.  
> When in doubt, identity confidence decreases — never increases.

---

## Identity Signals

All identity sources are normalized into a common structure:

```json
{
  "source": "voice | vision | satellite | explicit",
  "user_id": "user_a | unknown",
  "confidence": 0.0
}
```

Sources:
- `voice`: speaker embeddings
- `vision`: face recognition
- `satellite`: local inference on device
- `explicit`: user self-identification (“I am User_A”)

---

## Signal Trust Weights (Suggested Defaults)

| Source      | Weight | Notes |
|-------------|--------|-------|
| explicit    | 1.0    | Requires validation if risky |
| satellite   | 0.9    | Trusted hardware, configurable |
| vision      | 0.8    | Depends on image quality |
| voice       | 0.7    | Sensitive to noise |

Weights are configuration defaults, not guarantees.

---

## Fusion Strategy (High Level)

1. Collect all identity signals in a time window
2. Group signals by `user_id`
3. Compute weighted confidence per user
4. Detect conflicts
5. Apply identity lock rules
6. Output fused identity state

---

## Weighted Confidence Calculation

For each candidate user:

```text
confidence = Σ(signal.confidence × source.weight)
```

Normalization MAY be applied to keep values within [0,1].

---

## Identity Decision Thresholds (Suggested)

| State            | Threshold |
|------------------|-----------|
| CONFIRMED        | ≥ 0.85 |
| PROBABLE         | 0.60–0.84 |
| UNKNOWN          | < 0.60 |

Thresholds MUST be configurable.

---

## Conflict Detection

A conflict exists if:
- two different users both exceed PROBABLE threshold
- or a strong signal contradicts the locked identity

Example:
- voice → User_A (0.82)
- vision → User_B (0.88)

Result:
- identity_state = AMBIGUOUS
- system MUST ask clarification
- privacy restrictions increase

---

## Identity Lock Rules

### When Identity Lock Is Applied

- identity_state == CONFIRMED_ACTIVE
- no strong conflicting signals
- conversation is ongoing

### Effects of Identity Lock

- identity does NOT change automatically
- new signals are monitored but do not override
- conflicts downgrade state to AMBIGUOUS

---

## Multi-Speaker Scenarios

If multiple speakers are detected:
- each speaker has an independent identity context
- conversation mode MAY switch to SHARED
- private data disclosure is restricted

The system MUST NOT merge identities.

---

## Explicit Identity Claims

Example:
> “I am User_A”

Rules:
- creates an `explicit` identity signal
- MUST be validated against other signals
- MAY trigger verification if high-risk actions follow

Explicit claims alone do NOT guarantee identity.

---

## Satellite-Provided Identity

Satellites MAY send identity signals directly.

Rules:
- treated as high-confidence but not absolute
- conflicts with other sources downgrade confidence
- Core MAY override or ignore if inconsistent

---

## Vision + Voice Interaction Examples

### Case 1: Voice + Vision Agree
- voice → User_A (0.75)
- vision → User_A (0.90)

Result:
- CONFIRMED
- identity lock applies

---

### Case 2: Vision Only
- vision → unknown

Result:
- UNKNOWN
- ask for identification if needed

---

### Case 3: Voice Changes Mid-Conversation
- locked identity: User_A
- new voice signal: User_B (0.70)

Result:
- AMBIGUOUS
- system asks “Who is speaking now?”

---

## Privacy Interaction

- Identity fusion NEVER exposes profile data
- Identity uncertainty increases privacy restrictions
- Commands and disclosures are gated by identity state

---

## Failure Handling

If fusion fails:
- identity_state = UNKNOWN
- no commands executed
- no private data disclosed

---

## Summary

- Identity fusion is conservative and stateful
- Multiple signals are combined, not trusted individually
- Locks prevent identity flapping
- Conflicts reduce confidence
- Safety always wins over convenience

---

## Change Log

- 2026-01-16: Initial version.
