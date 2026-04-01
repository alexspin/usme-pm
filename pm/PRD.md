# PRD: Utility-Shaped Memory Ecology (USME)

**Status:** draft
**Last updated:** 2026-04-01
**Owner:** Alex

---

## Problem

OpenClaw's current context engine (lossless-claw / LCM) is **compression-first, history-preserving**.
Its job is to fit the full conversation history into the context window. It compresses — it does not curate.

**The core problem:** every turn, the model gets a compressed version of *everything*, regardless of relevance.
- A joke from two weeks ago gets the same weight as an architectural decision made yesterday.
- There is no episodic memory, no concept layer, no stable-facts store — just compressed chat history.
- Nothing learns. Every session starts fresh on "what matters."

**USME reframes the question:** instead of *"how do we preserve history?"*, the question is *"what should the model know right now to do its best work?"*

That is a harder question — and a significantly more valuable one.

---

## Goals

- G1: Per-turn context window that reflects **what matters for this task**, not just what's most recent
- G2: A **selection policy** that improves over time — the agent gets better as it accumulates experience
- G3: **Skill distillation** — high-value recurring interactions automatically become SKILL.md bundles
- G4: A **memory critic / editorial layer** that keeps stale, superseded, and low-provenance items out of context
- G5: Ship as an OpenClaw Context Engine plugin — a drop-in replacement for lossless-claw

---

## Non-Goals (v1)

- Full Postgres + pgvector stack (use SQLite + vector extension in v1)
- Multi-tenant / multi-user memory isolation (single-user scope for v1)
- Online RL / bandit-based policy learning (formula-based scoring in v1; ML is v2)
- HIPAA/SOC2/compliance instrumentation
- Governance / permissions layer (SpiceDB, OPA)
- ClawHub skill publishing pipeline

---

## Success Metrics

| Metric | Baseline | Target | Notes |
|---|---|---|---|
| Context tokens per turn | ~90K (post-distiller) | ≤60K | Tighter selection = smaller context |
| Context relevance | subjective | Measurably better (A/B eval) | Q: define eval harness |
| Skills auto-generated / week | 0 | ≥1 usable skill/week | FR-08 working |
| Stale memory in context | unknown | <5% of items | Memory critic working |
| Hot path latency | ~0ms (pass-through) | P95 ≤150ms | Local-first path |

*(Success metrics are provisional — see OPEN_QUESTIONS.md Q2)*

---

## Scope

### In Scope (v1)
- Multi-timescale memory stores: sensory trace, episodic store, concept layer (FR-02)
- Per-turn context packaging (FR-01): returns ordered `messages` list + `estimatedTokens` + optional `systemPromptAddition`
- Multi-mode packaging: `tight` and `extended` modes (FR-04)
- Selection policy v1: weighted formula (semantic similarity + recency decay + provenance tier + access frequency) (FR-03)
- Memory critic v1: editorial scoring layer that filters and ranks candidate memory items (FR-05)
- Consolidation job: nightly summarize, dedupe, promote stable facts, decay stale items (FR-06)
- Overnight skill distillation: rank high-value interactions → promote to SKILL.md bundles (FR-08)
- OpenClaw plugin packaging in `plugins.slots.contextEngine` slot (FR-09)
- SQLite + sqlite-vec (or equivalent) as storage backend

### Out of Scope (v1)
- Online learning hooks / OPE evaluation (FR-07) — v2
- Postgres + pgvector migration — deferred to v2 or until SQLite hits limits
- Memory permissions / governance layer — deferred
- Multi-agent memory sharing (Adam's agent, other agents) — deferred

---

## Constraints

- **Tech:** must integrate via `plugins.slots.contextEngine` — lossless-claw is the incumbent to replace
- **Latency:** hot path must be non-blocking; async work happens in the background
- **Ops:** zero new service dependencies in v1 — SQLite only, no Postgres
- **Compatibility:** graceful degradation — always fall back to full LCM context if USME fails

---

## Architecture Summary

USME is a **two-plane system**:

**Hot path (per turn, low latency):**
```
query → retrieve candidates → score (similarity + decay + provenance) → critic gate → package (tight/extended) → model
```

**Async path (overnight):**
```
consolidate → dedupe → promote facts → decay stale → distill skills → update utility stats
```

Integration point: `plugins.slots.contextEngine` — replaces lossless-claw.
See DECISIONS.md ADR-001 for rationale.

### Memory Store Layers
| Layer | What it holds | Timescale |
|---|---|---|
| Sensory trace | Raw turn content (tool calls, messages) | Hours → days |
| Episodic store | Compressed episode summaries | Days → weeks |
| Concept layer | Stable facts, preferences, decisions | Indefinite |
| Working-set constructor | Per-turn assembly from above layers | Per turn |

### Memory Item (key fields)
- `type`: sensory_trace | episode | fact | preference | plan | skill_candidate | tool_result
- `provenance`: source_kind (user | tool | web | file | model), source_ref, extractor_version
- `safety.prompt_injection_risk_score` — critic uses this (multi-user contexts)
- `utility`: prior, posterior_mean, decay_half_life — updated by async path
- `links`: graph edges (causes, supports, contradicts, refines, supersedes)

---

## Open Questions

See OPEN_QUESTIONS.md for the live list.

---

## Revision History

| Date | Author | Change |
|---|---|---|
| 2026-04-01 | Rufus | Initial draft from spec PDF + transcript analysis |
