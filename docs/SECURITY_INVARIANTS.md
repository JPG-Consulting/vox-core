# SECURITY_INVARIANTS.md

## Purpose

This document defines the **non-negotiable security and privacy rules** of the system.

These rules:
- MUST ALWAYS be enforced by the core/server.
- MUST NOT be overridden by prompts, models, or UI logic.
- Take precedence over all other documentation.

If any other document, code, or AI-generated change conflicts with this document,
**this document wins**.

---

## Fundamental Principle

> **When in doubt, do nothing.**  
> Fail closed, never fail open.

If identity, consent, authorization, or intent is unclear:
- Do NOT execute commands
- Do NOT disclose private data
- Ask for clarification or confirmation


---

## Documentation Safety Invariants

### INV-DOC-1: Invariant Integrity

- This document is governed by `DOCUMENTATION_POLICY.md`.
- Invariants defined here MUST NOT be weakened, removed, or reinterpreted
  by documentation changes that violate the additive-only rule.
- If a documentation change conflicts with this document, the change MUST be rejected.

---

## Identity Invariants

### INV-IDENTITY-1: Identity Is Probabilistic

- Identity MUST be treated as probabilistic and stateful.
- Identity MUST NOT be assumed from a single signal.
- Voice, image, satellite-provided identity, and user statements are signals, not facts.

---

### INV-IDENTITY-2: Identity Is Sticky During Active Conversation

- Once an identity is CONFIRMED and active, it MUST remain locked
  unless strong evidence indicates otherwise.
- Identity MUST NOT change automatically due to weak or ambiguous signals.

---

### INV-IDENTITY-3: No Implicit Identity Escalation

- The system MUST NOT escalate identity confidence automatically.
- Confirmation, verification, or explicit user action is required.

---

## Age & Child Safety Invariants

### INV-AGE-1: Age-Aware Behavior

- The system MUST take the user’s age into account when selecting tone, content, and safety behavior.
- Age MUST be derived from birthdate when possible.

### INV-AGE-2: Unknown Age Defaults to Minor-Safe

- If the user’s age group is `unknown`, the system MUST behave conservatively as if the user is a minor.
- In minor-safe mode, the system MUST avoid adult topics and MUST apply stricter command and disclosure rules.

### INV-AGE-3: Minors MUST NOT Self-Modify Age or Birthdate

- If a user is classified as a minor, the system MUST NOT allow that user to:
  - change their birthdate
  - delete their birthdate
  - mark their birthdate as unknown
  - otherwise alter age-relevant profile fields
- Any such attempt MUST be rejected.
- The system MUST return a warning explaining that the operation is restricted
  for safety reasons.

### INV-AGE-4: Birthdate Modification Authority and Auditability

- Birthdate changes are permitted only if:
  - the profile owner is classified as an adult, OR
  - an authorized adult modifies a minor’s birthdate under an explicit policy.
- All birthdate changes MUST be auditable, including:
  - who performed the change
  - when it occurred
  - the reason or authority under which it was performed.

## Privacy Invariants

### INV-PRIVACY-1: Private Data Is Private by Default

- All user profile data is PRIVATE unless explicitly marked otherwise.
- Private data MUST NOT be disclosed to other users without consent.

---

### INV-PRIVACY-NAME-1: Legal Name Protection

- A user’s legal/real name (identity name) MUST NOT be used in casual interaction.
- The system SHOULD address users using their `preferred_name` when available.
- Legal names MAY be used only for:
  - safety-critical dialogs
  - integrations requiring identity certainty
  - internal logs


---

## Ownership and Preference Immutability Invariants

### INV-OWN-1: Profile Ownership Is Exclusive

- A user profile has exactly one owner.
- Only the profile owner may authorize permanent changes to their own profile data.
- Roles, delegation, or command authority MUST NOT grant the ability to modify
  another user’s profile ownership or preferences.

### INV-OWN-2: Addressing Preferences Are Owner-Only

- Addressing preferences, including:
  - preferred name or nickname
  - grammatical gender or form-of-address preferences
  - language preference for interaction
  MUST be modifiable only by the profile owner.
- The system MUST NOT infer, override, or update these fields based on
  third-party statements or conversational inference.

### INV-OWN-3: Identity Fields Are Immutable to Third Parties

- Identity-defining fields (including legal name and preferred name) MUST NOT be
  modified by any party other than the profile owner.
- Age-relevant attributes follow the Age & Child Safety invariants in this document.
- Any attempt to modify another user’s identity fields MUST be denied (fail closed).

---

### INV-PRIVACY-2: Data Sources MUST NOT Be Mixed

The system distinguishes at least:
- Conversation memory
- User profile memory

Rules:
- Conversation memory MAY be shared among participants of that conversation.
- User profile data MUST NOT be used to answer questions for other users
  unless explicit consent exists.

---

### INV-PRIVACY-FACT-1: Profile Owner Precedence

- Personal facts stored in a profile belong to the profile owner.
- If the profile owner provides a value for a fact, it is authoritative.
- Third-party facts MUST NOT override owner-provided facts.

### INV-PRIVACY-FACT-2: Third-Party Visibility Is Limited


---

## Learning and Social-Engineering Resistance Invariants

### INV-LEARN-1: Learning Does Not Imply Authority

- The system MAY learn patterns or observations from interaction.
- Learning MUST NOT grant authority to modify profile data or preferences.
- Learned information MUST NOT bypass ownership, consent, or age restrictions.

### INV-LEARN-2: No Indirect Profile Modification

- The system MUST NOT accept indirect requests to modify a user profile,
  including attempts that reference prior statements, implied consent,
  or third-party assertions.
- Profile modifications MUST originate from the profile owner through
  an explicit, validated update flow.

### INV-LEARN-3: Third-Party Facts Are Additive-Only and Non-Authoritative

- The system MAY store third-party statements about another user as
  non-authoritative claims with provenance and confidence.
- Third-party claims MUST be additive-only:
  - they MAY add new candidate facts with provenance
  - they MUST NOT modify or delete owner-provided facts
  - they MUST NOT modify preferences, identity fields, or age-relevant attributes

### INV-LEARN-4: No Authority Escalation by Repetition or Pressure

- Repeated statements, conversational pressure, or persistence MUST NOT:
  - increase authority
  - relax security constraints
  - override confirmation requirements
- Authority and permissions MUST be evaluated solely from validated identity,
  role, and delegation state.

### INV-LEARN-5: Conflict Handling MUST Protect the Owner

- If a third-party claim conflicts with an owner-provided value:
  - the owner-provided value MUST remain authoritative
  - the conflicting claim MUST NOT cause disclosure of the owner’s value
    to the third-party provider
  - the system SHOULD avoid revealing that a conflict exists, unless
    the profile owner is the requester

- If a user provides a fact about another user, they MAY be allowed to access that fact **only** within the visibility rules recorded for that fact.
- If the profile owner has provided a conflicting value, the owner’s value MUST NOT be disclosed to the third-party provider.

---

### INV-PRIVACY-3: Ownership Allows Self-Disclosure Only

- A user MAY access their own private data if:
  - their identity is CONFIRMED
  - the request context is safe (see conversation modes)
- In shared conversations, conservative policy applies:
  - private data disclosure requires explicit confirmation

---

### INV-PRIVACY-4: Consent Must Be Explicit

- Consent MUST be:
  - explicitly stated
  - attributable to the data owner
  - scoped (what data, who can receive it)
  - temporary (conversation or time-bounded)

- Consent MUST NOT be:
  - inferred
  - assumed
  - permanent by default

---

## Conversation Mode Invariants

### INV-CONV-1: Conversation Mode Controls Disclosure

Conversation mode determines what may be said, not who is speaking.

At minimum:
- PRIVATE
- SHARED_VERIFIED
- SHARED_UNVERIFIED

Rules:
- In SHARED modes, private data MUST NOT be disclosed
  unless consent or ownership rules allow it.

---

### INV-CONV-2: Mode Changes Are Explicit

- Conversation mode MUST NOT change implicitly.
- Mode changes require:
  - explicit user statement
  - or strong, verified detection (e.g., confirmed new participant)

---


---

## Date & System Event Invariants

### INV-CONV-BDAY-1: Birthday Greeting Safety

- The system MAY greet a user for their birthday only if:
  - identity is CONFIRMED
  - current date is known/trusted
  - birthdate is known with sufficient confidence
  - the greeting has not already occurred for that user on that date
- Birthday greetings MUST NOT disclose sensitive data (e.g., full birthdate) unless allowed.

## Command Authorization Invariants

### INV-CMD-1: Commands Require Authorization

- Every command MUST define required permissions.
- A command MUST NOT execute unless the requester is authorized.

Knowing someone ≠ being authorized.

---

### INV-CMD-2: Delegation Is Explicit and Limited

- Delegation MUST be:
  - granted explicitly
  - granted by a user with sufficient authority
  - scoped (permissions, duration, context)

- Delegation MUST NOT:
  - grant more permissions than the grantor has
  - allow re-delegation unless explicitly allowed

---

### INV-CMD-3: No Permission Without Confirmation

- Granting permissions MUST use a two-phase flow:
  1. detect intent
  2. explicit confirmation by the owner

Ambiguous phrases MUST NOT grant permissions.

---


---

## Command Group Authorization Invariants

### INV-CMD-GRP-1: Commands Are Authorized by Group

- Commands MUST belong to one or more explicit command groups.
- Authorization MUST be evaluated at the command-group level, not per-command
  in isolation.

### INV-CMD-GRP-2: Command Group Permissions Are Non-Transitive

- Authorization for one command group MUST NOT imply authorization
  for any other command group.
- Delegation of a command group MUST NOT grant authority outside that group.

### INV-CMD-GRP-3: Group Authorization Is Required for Execution

- A command MUST NOT be executed unless the requesting identity is authorized
  for the command’s group and all confirmation requirements are satisfied.

## Command Confirmation Invariants

### INV-CMD-4: Risky Commands Require Confirmation

Commands classified as risky MUST:
- require explicit confirmation
- be executed only after confirmation
- expire if confirmation is not received in time

---

### INV-CMD-5: Confirmation Must Come From the Requester

- Confirmation MUST come from:
  - the same identity that requested the command
  - with identity still confirmed
- Confirmations from other users MUST be ignored.

---

## Multi-User Safety Invariants

### INV-MULTI-1: Presence Increases Restrictions

- When multiple users are present:
  - privacy rules become stricter
  - confirmations become mandatory for sensitive actions

---

### INV-MULTI-2: No Cross-User Leakage

- The system MUST NOT:
  - reveal one user’s private data to another
  - infer permissions based on social presence

---

## LLM Usage Invariants

### INV-LLM-1: LLM Is Not a Policy Engine

- The LLM MUST NOT:
  - decide authorization
  - decide consent
  - decide identity
  - bypass confirmations

All enforcement MUST happen in the core.

---

### INV-LLM-2: Context Filtering Is Mandatory

- The LLM MUST only receive:
  - data that is explicitly allowed
  - no hidden or excess context

If the LLM does not see it, it cannot leak it.

---

## Failure Handling Invariants

### INV-FAIL-1: Fail Closed

If any of the following fail:
- STT
- identity resolution
- permission checks
- confirmation logic

Then:
- commands MUST NOT execute
- private data MUST NOT be disclosed

---

### INV-FAIL-2: Errors Must Be User-Safe

On failure, the system SHOULD:
- apologize briefly
- ask to repeat or clarify
- reset pending states safely

---

## Summary (Authoritative)

1. Identity is never assumed
2. Privacy is default
3. Consent is explicit
4. Authorization is mandatory
5. Risky actions require confirmation
6. Ambiguity results in no action
7. The core enforces all rules
8. The LLM only generates text

---

## Change Log

- 2026-01-16: Initial version.
- 2026-01-16: Added age safety, legal-name protection, third-party fact rules, and birthday greeting safety.
