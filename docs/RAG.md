# RAG.md

## Purpose

This document defines the **Retrieval-Augmented Generation (RAG)** strategy of the project.

RAG is used to:
- answer questions beyond the local model’s knowledge
- provide fallback to external LLMs or APIs
- store verified knowledge for future use

RAG MUST respect all privacy, consent, and security invariants.

This document depends on:
- PROJECT_OVERVIEW.md
- SECURITY_INVARIANTS.md
- ARCHITECTURE.md
- DATA_MODELS.md
- INTENTS.md

---

## Scope

In scope:
- fallback to external knowledge sources
- retrieval of stored knowledge
- controlled storage of new information
- provenance tracking

Out of scope (initially):
- autonomous web crawling
- unsupervised knowledge ingestion
- personal data mining

---

## RAG Architecture (High Level)

```text
User Question
   ↓
Intent Detection
   ↓
Local Knowledge Check
   ↓
[ if insufficient ]
   ↓
External Knowledge Source (LLM / API)
   ↓
Response
   ↓
Optional Storage (with rules)
```

RAG is orchestrated by the Core, not the LLM.

---

## Knowledge Sources

### 1) Local Knowledge Store

- Structured or semi-structured data
- Retrieved by embeddings, keywords, or metadata
- Stored with provenance and confidence

Example entry:

```json
{
  "key": "capital_of_france",
  "value": "Paris",
  "source": "external_llm",
  "verified": false,
  "created_at": "2026-01-16"
}
```

---

### 2) External Knowledge Providers

Examples:
- OpenAI
- Gemini
- Other LLM APIs
- Specialized APIs (weather, finance, etc.)

Rules:
- External providers are fallback-only
- Requests MUST be filtered for privacy
- No private user data may be sent unless explicitly allowed

---

## Privacy and Consent Rules

### RAG-PRIV-1: No Private Data Leakage

- User profile data MUST NOT be sent to external providers
- Conversation memory MUST be sanitized
- Identity MUST be anonymized unless explicitly required

---

### RAG-PRIV-2: Explicit Consent for Learning

Storing new information from a conversation:
- MUST NOT include private personal data
- MUST require explicit consent if ambiguous
- MUST respect data ownership

Example:
> “Do you want me to remember this for the future?”

---

## Storage Policy

Knowledge MAY be stored only if:
- it is non-personal OR explicitly allowed
- it has clear provenance
- it does not violate SECURITY_INVARIANTS.md

Knowledge MUST include:
- source
- confidence
- scope (global / local / conversation-derived)

---

## Retrieval Rules

When answering a question:
1. Prefer local knowledge
2. Use RAG only if insufficient
3. Prefer verified knowledge
4. Clearly indicate uncertainty if present

The system SHOULD NOT hallucinate missing facts.

---

## Interaction with Intents

RAG is typically triggered by:
- informational intents
- “I don’t know” situations
- explicit user requests for lookup

RAG MUST NOT be triggered for:
- commands
- permission changes
- identity resolution

---

## Fallback Behavior

If all knowledge sources fail:
- respond safely
- acknowledge uncertainty
- do not invent answers

Example:
> “I don’t have that information right now.”

---

## Future Extensions (Planned)

- verification workflows
- human-in-the-loop validation
- knowledge expiration policies
- per-user or per-domain knowledge scopes

---

## Summary

- RAG is controlled and conservative
- External LLMs are fallbacks, not authorities
- Knowledge storage is explicit and consent-aware
- Privacy rules always apply

---

## Change Log

- 2026-01-16: Initial version.
