# PaulAI — Vision and Scope

Oct 16, 2026 · @Joe D G · Draft v0.1

## Purpose

PaulAI gives a single user one front end to ChatGPT and Claude, capturing every prompt and response, importing history from those tools, and enabling search and inference across that combined history, solving the problem that no individual AI can see across a user's conversations, projects, and providers.

PaulAI is named for an old friend: insightful, with an excellent memory, and funny.

## Problem and context

Each AI tool keeps its own history in silos. ChatGPT cannot see across its own projects and conversations, and no tool can see across providers.

- Decisions, ideas, and advice are scattered across many conversations and two providers.
- There is no way to ask questions of that history as a whole, such as what was decided about a topic, or where providers disagreed.
- Consumer chat apps cannot be captured live; their history is reachable only through periodic exports.

## Scope

Release 1 is a single-user app covering live chat, capture, import, search, and inference across ChatGPT and Claude.

**In scope**

- One user; no multi-user accounts or sharing
- Live chat with ChatGPT and Claude through their application programming interfaces (APIs)
- Capture of every prompt and response, with metadata and the raw payload
- Import of existing history from each provider's export
- Keyword and semantic search across all history
- Inferential queries across conversations, projects, and providers, with cited sources
- PaulAI's persona: a consistent voice that draws on relevant history
- Token and cost tracking per provider

**Out of scope for release 1**

- Multi-user support
- Providers beyond the two named, including Gemini (a candidate for the O8 extensibility test)
- Live capture of consumer chat apps (web or mobile); these come in only by import
- Images, audio, and file attachments, beyond recording that they existed (open question)
- Mobile app; access is through the web front end

## Objectives

Nine objectives, in priority order; inference (O5) is the reason PaulAI exists and the hardest to verify.

| # | Objective | Success criterion |
| --- | --- | --- |
| O1 | Unified front end: chat with both providers from one interface | Text chat with streaming works for ChatGPT and Claude |
| O2 | Complete capture of every prompt and response, with metadata and raw payload | Zero lost exchanges, verified by periodic reconciliation |
| O3 | History import from both providers | All exported conversations imported; re-import creates no duplicates |
| O4 | Keyword and semantic search across all history | Relevant result in the top 5 for a test query set, in under 2 seconds |
| O5 | Inference across conversations, projects, and providers | At least 80% correct on ~20 real test questions; every answer cites its source conversations |
| O6 | Privacy and security: data under the user's control | No plaintext API keys; history stored locally or encrypted |
| O7 | Cost visibility: tokens and cost per provider | Monthly totals within 5% of provider bills |
| O8 | Extensibility: new providers by adapter or configuration | A third provider (such as Gemini) added without changing core code |
| O9 | Persona: consistent voice, insightful and funny, drawing on history | User's subjective review |

Persona is ranked last so the name does not drive priority.

## Constraints and non-functional requirements

External formats and APIs are fixed inputs; part of the data model is dictated by them.

**Constraints**

- Provider APIs and export formats are controlled by OpenAI and Anthropic and may change without notice.
- ChatGPT and Claude exports are JSON (JavaScript Object Notation). Both formats must be verified when the exports arrive.
- Open-source components only; no Oracle or other licensed stack.
- Single developer, using Claude as a collaborator.

**Non-functional requirements (to be quantified during analysis)**

- Privacy: history stays on hardware the user controls.
- Security: API keys never stored in plaintext.
- Latency: search under 2 seconds; chat adds no noticeable delay over calling a provider directly.
- Durability: no lost exchanges; regular backups.
- Cost: token spend visible and bounded.
- Retention: keep all history indefinitely unless deleted by the user.

## Candidate architecture and technology

PaulAI is a layered proxy: every provider, store, and model sits behind an adapter (ports and adapters, or hexagonal, pattern), so each can be swapped without changing the core.

> *Diagram (candidate architecture: 3 layers, adapters at the edges) lives in the [live doc](https://claude.ai/artifact/M2B4mw9h4YEd4EryhRKvNn).*

The query and inference engine is the core of PaulAI; it combines SQL (Structured Query Language), vector search, and large language model (LLM) reasoning. Raw provider payloads are stored alongside the normalized form so no provider-specific data is lost.

**Candidate technology (working assumptions, to be confirmed by the architecturally significant use cases)**

- Back end: Java (decided; chosen for the developer's decades of Java experience, so generated code can be reviewed fluently)
- Provider layer: Spring AI or LangChain4j, to be evaluated; both support OpenAI, Anthropic, embeddings, and pgvector (no custom provider abstraction)
- Storage: PostgreSQL with the pgvector extension; a graph store only if the inference prototype shows a need
- Deployment: Docker, hosted locally
- Front end: web; framework open

## Risks

The biggest risk is building a good logger that cannot answer the questions that matter.

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Inference quality falls short of O5 | PaulAI's core value fails | Time-boxed inference prototype (spike) during analysis, tested against the ~20 real questions |
| Data model cannot support inference | Costly rework after design | Prototype inference before locking the data model |
| Provider APIs or export formats change | Capture or import breaks | Isolate each provider behind an adapter; store raw payloads |
| Leaky AI abstraction loses provider-specific data | Lost tool use, reasoning, or attachments | Store raw request and response alongside the normalized form |
| Persona work expands scope | Delays core objectives | Persona ranked last (O9) |
| Waterfall drift: all analysis before any design | Late discovery of architectural problems | Iterate one architecturally significant use case at a time |

## Open questions and next steps

**Open questions**

- What exactly counts as an "inferential query"? Needs a precise definition in the glossary.
- Are attachments (images, files) in scope beyond recording that they existed?
- Which machine hosts PaulAI, and is remote access needed away from home?
- How often are consumer-app histories re-imported?

**Next steps**

- [ ] Request ChatGPT and Claude exports
- [ ] Write ~20 real questions to ask of the combined history
- [ ] Draft a glossary of key terms
- [ ] Define use cases and identify the architecturally significant ones
- [ ] Capture the data model, bottom-up from the exports and top-down from the use cases
- [ ] Run the inference prototype
- [x] Set up the git repository
