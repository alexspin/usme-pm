# Decisions: USME

Key decisions logged in ADR style.

---

## ADR-001: Integration point is the contextEngine slot — USME replaces lossless-claw

**Date:** 2026-03-30
**Status:** accepted
**Decided by:** Alex + Rufus (transcript 2026-03-30)

**Context:**
USME needs a place to hook into OpenClaw's run loop. Two options: (1) use the `before_prompt_build` hook (what the distiller used), or (2) claim the `plugins.slots.contextEngine` slot.

**Options considered:**
1. `before_prompt_build` hook — additive, non-destructive, doesn't replace LCM
2. `plugins.slots.contextEngine` — direct replacement of lossless-claw's context engine

**Decision:**
Option 2. USME plugs into `plugins.slots.contextEngine` as a replacement for lossless-claw. The spec explicitly designs for this integration point and it's the only path to actually controlling what goes into the context window (the hook cannot replace messages).

**Consequences:**
- This is a replacement, not an addition. lossless-claw will be disabled when USME is active.
- USME must implement the full Context Engine contract: `ingest()`, `assemble()`, `compact()`
- Migration strategy needed: can we run USME in shadow mode first? (see OPEN_QUESTIONS.md Q4)

---

## ADR-002: Memory critic framing — editorial layer, not security guard

**Date:** 2026-03-30
**Status:** accepted
**Decided by:** Alex (transcript 2026-03-30)

**Context:**
The spec frames the memory critic (FR-05) primarily as a security tool — preventing memory injection attacks from untrusted users. Alex reframed it as a quality tool.

**Options considered:**
1. Security-first framing: block injections, flag suspicious items (relevant for multi-tenant SaaS)
2. Editorial-first framing: evaluate relevance, recency, and provenance to curate what enters context

**Decision:**
Editorial-first (Option 2) for v1. We are single-user, self-hosted — injection attacks are not the threat model. The critic's job is to ask "should this actually be here right now?" based on:
- Is it still true? (recency/decay)
- Is it relevant to the current query? (semantic match)
- Is it signal or noise? (promote resolution facts, bury debugging traces)
- How confident is this? (provenance tier: explicit statement > tool result > inference)
- Has it been superseded? (contradiction detection)

**Consequences:**
- Simpler threat model for v1; security layer can be added later for multi-user scenarios
- Critic must implement contradiction detection (memory A supersedes memory B)
- Provenance tiers must be defined and enforced at ingest time

---

## ADR-003: V1 storage — SQLite + vector extension (not Postgres + pgvector)

**Date:** 2026-03-30
**Status:** accepted
**Decided by:** Rufus recommendation, Alex agreed (implicit)

**Context:**
The spec recommends Postgres + pgvector as the default storage. We currently run zero databases on the server.

**Options considered:**
1. Postgres + pgvector — production-grade, strong transactional semantics, more ops complexity
2. SQLite + sqlite-vec (or similar) — zero new service dependencies, simpler ops, sufficient for single-user

**Decision:**
SQLite + vector extension for v1. Single-user workload does not justify Postgres ops complexity. SQLite has proven sufficient (lossless-claw uses it today). Migrate to Postgres if/when we hit limits.

**Consequences:**
- Must select a SQLite vector extension: candidates are `sqlite-vec`, `sqlite-vss`, `vectorlite`
- HNSW vs IVFFlat index choice deferred until extension is selected
- Schema must be migration-friendly for future Postgres move

---

## ADR-004: V1 selection policy — weighted formula, not RL bandit

**Date:** 2026-03-30
**Status:** accepted
**Decided by:** Rufus recommendation, Alex agreed (implicit)

**Context:**
FR-03 specifies a "learned retrieval + selection policy" using contextual bandits / RL-style credit assignment. Building a full bandit requires labeled training data we don't have.

**Options considered:**
1. Full RL bandit (FR-07 machinery) — best eventual quality, requires feedback loops and training data
2. Weighted formula — deterministic, tunable, ships immediately, no training data needed

**Decision:**
Weighted formula for v1 with four factors:
1. Semantic relevance to current query (vector similarity score)
2. Recency / decay (exponential decay from `last_confirmed_at`)
3. Provenance tier (explicit user statement = 1.0 > tool result = 0.7 > model inference = 0.4)
4. Access frequency (retrieval rate as a proxy for utility)

Items below an inclusion threshold are excluded. Items above are ranked and packed until token budget is exhausted.

V2 adds online learning (FR-07): reward signals when model performs well → update formula weights.

**Consequences:**
- Must define and tune the formula weights empirically
- Need a way to emit "this memory was useful / not useful" signals for future v2 learning
- Formula is transparent and auditable — useful for debugging retrieval quality

---

## ADR-005: Build slim v1 — FR-03 + FR-04 + FR-08 first, not the full spec

**Date:** 2026-03-30
**Status:** accepted
**Decided by:** Rufus recommendation, Alex agreed (implicit)

**Context:**
The full spec (FR-01 to FR-09) is a 4-6 week build before any of the interesting capabilities ship. Must prioritize.

**Options considered:**
1. Build full spec end-to-end before shipping anything
2. Identify the 2-3 FRs with highest leverage and ship those first

**Decision:**
Build slim v1 targeting:
- **FR-03 + FR-04** (selection policy + multi-mode packaging) — core value proposition; makes every conversation noticeably better
- **FR-08** (overnight skill distillation) — most novel capability; most aligned with "organizational memory that compounds" pitch

Full FR list remains the roadmap; everything else follows once v1 is validated.

**Consequences:**
- FR-07 (online learning) is explicitly deferred — need v1 shipped first to collect data
- FR-05 (memory critic) is included in v1 but scoped to editorial scoring, not full security apparatus
- FR-06 (consolidation/sleep) is required for v1 to prevent unbounded memory growth
