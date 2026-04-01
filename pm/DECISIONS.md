# Decisions: USME

Key decisions logged in ADR style.

---

## ADR-006: Ports and adapters — usme-core library + openclaw adapter

**Date:** 2026-04-01
**Status:** accepted
**Decided by:** Alex

**Context:**
USME needs to live in rufus-plugin (primary home) but also be usable in other agent frameworks (LangChain, CrewAI, custom harnesses). These are in tension if the core logic knows about OpenClaw.

**Options considered:**
1. Build directly as an OpenClaw plugin — simple, but locked in; can't be reused elsewhere
2. Build as a standalone package only — more portable, but disconnected from rufus-plugin
3. Ports and adapters: framework-agnostic core + thin OpenClaw adapter wrapper

**Decision:**
Option 3. Two layers with a hard boundary:
- **`usme-core`**: pure TypeScript npm package, zero OpenClaw imports. Owns storage, selection policy, memory critic, consolidation, skill distillation. Public interface: `ingest()`, `assemble()`, `consolidate()`, `distillSkills()`. Lives at `rufus-projects/usme/`.
- **OpenClaw adapter**: thin wrapper (~100 lines) inside rufus-plugin. Implements OpenClaw's Context Engine contract, maps turn format → MemoryItem, registers via `plugins.slots.contextEngine`.

Future adapters (LangChain, CrewAI, etc.) implement their own interface on top of `usme-core`.

**The rule:** nothing in `usme-core` imports from openclaw. The adapter knows both; the core knows neither.

**Consequences:**
- Core and adapter can be built in parallel once the interface is locked
- `usme-core` is publishable to npm independently if/when that's useful
- Interface contract must be defined and locked before adapter work begins

---

## ADR-008: Storage — sqlite-vec + node:sqlite built-in (Node v25)

**Date:** 2026-04-01
**Status:** accepted
**Decided by:** Rufus (research) — confirmed by Q5/Q6 constraints

**Context:**
ADR-003 selected SQLite + vector extension. Need to pick the specific extension and binding.

**Options considered:**
1. `better-sqlite3` + `sqlite-vec` — synchronous, fast, but native build issues on this box (documented in TOOLS.md)
2. `node:sqlite` (built-in Node v25) + `sqlite-vec` — synchronous, built-in, no native compilation, `loadExtension()` supported
3. LanceDB (`@lancedb/lancedb`) — embedded, Rust-based native binary per platform, async API, larger scope

**Decision:**
`node:sqlite` (built-in) + `sqlite-vec` npm package.
- Node v25.8.1 (our runtime) ships `node:sqlite` with `loadExtension()` support
- `sqlite-vec` is pure C — loads cleanly, no compilation step, no platform issues
- Synchronous in-process path = lowest possible hot path latency (no async overhead, no IPC)
- LanceDB rejected: Rust native binary has platform load risks (documented Windows issue), async API adds latency, billion-scale features are overkill for single-user

**Consequences:**
- Zero new native dependencies — no node-gyp, no better-sqlite3
- Must validate `node:sqlite` + `sqlite-vec` integration in a prototype before starting schema design
- HNSW index via sqlite-vec for ANN search; flat scan acceptable for small corpora (<10K memories)

---

## ADR-007: Transition — shadow mode first, hard cutover after validation

**Date:** 2026-04-01
**Status:** accepted
**Decided by:** Alex

**Context:**
LCM (lossless-claw) is working in production today. USME replaces it. Need a safe cutover strategy.

**Options considered:**
1. Hard cutover — disable LCM, enable USME, monitor
2. Shadow mode — USME runs in parallel, logs what it *would have* sent, compare vs LCM before enabling
3. Gradual — USME for new sessions, LCM for existing

**Decision:**
Shadow mode first (Option 2). USME runs alongside LCM, assembles context each turn but does not send it to the model — logs output to JSONL for comparison. Hard cutover only after shadow evaluation confirms USME quality ≥ LCM.

**Consequences:**
- Must build a shadow mode flag into the OpenClaw adapter from day one
- Evaluation log format needed: per-turn JSONL with {turn_id, lcm_tokens, usme_tokens, usme_top_items, timestamp}
- Shadow mode is also the natural A/B harness for measuring token savings (Q2 success metric)

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
