# Roadmap: USME

**Last updated:** 2026-04-01

Legend: `[x]` done · `[-]` in progress · `[ ]` not started

---

## Phase 0: Foundation (Discovery + Design — 2026-04)

Answer the 6 open questions and lock the design before writing code.

- [ ] Q1–Q6: Answer all open discovery questions (see OPEN_QUESTIONS.md)
- [ ] PRD approved — move to DESIGN
- [ ] Write architecture.md: data model, storage schema, hot path flow, async path flow
- [x] Storage decision locked: Postgres + TimescaleDB + pgvector (ADR-009)
- [ ] Decide: shadow mode rollout vs hard cutover

---

## Phase 1: Core Context Engine — v1 (Build — 2026-05)

The minimum useful USME: a context engine that selects by relevance, not just recency.

- [ ] **M1.1: Storage layer**
  - [ ] Postgres + TimescaleDB + pgvector Docker compose setup
  - [ ] Schema: sensory_trace (standard), episodes (hypertable), concepts (pgvector HNSW), skills (standard)
  - [ ] node-pg-migrate setup for schema versioning
  - [ ] Basic CRUD + vector index maintenance

- [ ] **M1.2: Selection policy v1 (FR-03)**
  - [ ] Weighted formula: semantic similarity + decay + provenance + access freq
  - [ ] Token budget enforcement: pack items until budget exhausted
  - [ ] Configurable weights in plugin config

- [ ] **M1.3: Multi-mode packaging (FR-04)**
  - [ ] `tight` mode: high-relevance only, strict token budget
  - [ ] `extended` mode: broader recall, relaxed budget
  - [ ] Mode selectable via runtime hint from agent/tooling

- [ ] **M1.4: Async Haiku/Flash extractor (ADR-011)**
  - [ ] Per-turn extraction: fires async after response delivered
  - [ ] Structured output: {type, content, provenance, utility_estimate, tags[]}
  - [ ] In-process task queue; graceful degradation if extractor fails

- [ ] **M1.5: Memory critic v1 (FR-05 — editorial)**
  - [ ] Recency/decay scoring
  - [ ] Supersession detection (new fact overrides old fact)
  - [ ] Provenance tiering at ingest

- [ ] **M1.6: OpenClaw plugin packaging (FR-09)**
  - [ ] Implements Context Engine contract: ingest(), assemble(), compact()
  - [ ] Registered via plugins.slots.contextEngine
  - [ ] Shadow mode flag: logs what USME would send vs what LCM sends

- [ ] **M1.7: Nightly consolidation + skill accrual (FR-06 + FR-08, ADR-012)**
  - [ ] Cluster sensory traces → episodes (pgvector cosine k-means + Sonnet compression)
  - [ ] Fact promotion: recurring episode patterns → concept layer (Sonnet adjudicates)
  - [ ] Contradiction resolution: semantic search for conflicts, Sonnet adjudicates
  - [ ] Skill candidate ranking: novelty × frequency × teachability formula
  - [ ] Skill drafting: Sonnet/Opus writes SKILL.md candidates for top-ranked interactions
  - [ ] Decay + prune: sensory traces TTL 30d; decay utility scores on stale concept items
  - [ ] Bounded growth check + alert if DB exceeds threshold

---

## Phase 2: Scale + Graph — v2 (2026-Q3)

Upgrade performance and add richer relationship traversal.

- [ ] **M2.1: pgvectorscale DiskANN indexes** — upgrade from HNSW for large collections
- [ ] **M2.2: Graph entity linking** — evaluate Neo4j or Memgraph for concept entity relationships (vs JSONB links)
- [ ] **M2.3: TimescaleDB continuous aggregates** — automated temporal roll-ups for episodic memory

---

## Phase 3: Learning Hooks — v3 (2026-Q4)

Add online learning on top of the v1 formula. Requires v1 to be running and collecting data.

- [ ] Feedback signal design: what constitutes a "good" response?
- [ ] Log decision + outcome to evaluation store (FR-07)
- [ ] OPE/DR evaluation harness: did retrieval policy improve outcomes?
- [ ] Update formula weights based on evaluation results
- [ ] Canary rollout for policy changes

---

## Phase 4: Multi-User + Publishing — v3+ (TBD)

- [ ] Multi-user memory isolation (permissions layer)
- [ ] Adam's agent integration — shared concept layer, separate episodic stores
- [ ] ClawHub skill publishing pipeline

---

## Icebox (future / unscheduled)

- [ ] Graph traversal on concept layer (contradiction resolution, lineage)
- [ ] Fast refiner (Flash/Haiku) in hot path for context condensation
- [ ] Memory audit UI in /rufus dashboard
- [ ] Export/import memory for cross-instance portability
- [ ] Benchmark harness (compare USME vs LCM on held-out eval set)

---

## Completed

- [x] USME spec PDF authored — 2026-03-30
- [x] Gap analysis vs LCM completed — 2026-03-30
- [x] PM docs initialized — 2026-04-01
