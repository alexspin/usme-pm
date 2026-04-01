# Roadmap: USME

**Last updated:** 2026-04-01

Legend: `[x]` done · `[-]` in progress · `[ ]` not started

---

## Phase 0: Foundation (Discovery + Design — 2026-04)

Answer the 6 open questions and lock the design before writing code.

- [ ] Q1–Q6: Answer all open discovery questions (see OPEN_QUESTIONS.md)
- [ ] PRD approved — move to DESIGN
- [ ] Write architecture.md: data model, storage schema, hot path flow, async path flow
- [ ] Select SQLite vector extension (leading candidate: sqlite-vec)
- [ ] Decide: shadow mode rollout vs hard cutover

---

## Phase 1: Core Context Engine — v1 (Build — 2026-05)

The minimum useful USME: a context engine that selects by relevance, not just recency.

- [ ] **M1.1: Storage layer**
  - [ ] SQLite + sqlite-vec schema (MemoryItem, MemoryLink tables)
  - [ ] Ingest pipeline: turn content → sensory trace + episode
  - [ ] Basic CRUD + vector index maintenance

- [ ] **M1.2: Selection policy v1 (FR-03)**
  - [ ] Weighted formula: semantic similarity + decay + provenance + access freq
  - [ ] Token budget enforcement: pack items until budget exhausted
  - [ ] Configurable weights in plugin config

- [ ] **M1.3: Multi-mode packaging (FR-04)**
  - [ ] `tight` mode: high-relevance only, strict token budget
  - [ ] `extended` mode: broader recall, relaxed budget
  - [ ] Mode selectable via runtime hint from agent/tooling

- [ ] **M1.4: Memory critic v1 (FR-05 — editorial)**
  - [ ] Recency/decay scoring
  - [ ] Supersession detection (new fact overrides old fact)
  - [ ] Provenance tiering at ingest

- [ ] **M1.5: OpenClaw plugin packaging (FR-09)**
  - [ ] Implements Context Engine contract: ingest(), assemble(), compact()
  - [ ] Registered via plugins.slots.contextEngine
  - [ ] Shadow mode flag: logs what USME would send vs what LCM sends

- [ ] **M1.6: Nightly consolidation (FR-06)**
  - [ ] Summarize sensory traces → episodes
  - [ ] Dedupe near-duplicate memories
  - [ ] Decay utility scores on stale items
  - [ ] Bounded growth check: alert if memory DB exceeds threshold

---

## Phase 2: Skill Distillation — v1 (Build — 2026-06)

The most novel capability. Turns daily usage into reusable SKILL.md bundles.

- [ ] **M2.1: Activity ranker**
  - [ ] Overnight job: scan previous day's episodes
  - [ ] Score by: novelty, frequency, recency, user engagement signals
  - [ ] Emit ranked list of candidate skill topics

- [ ] **M2.2: Skill evaluator**
  - [ ] Send top candidates to stronger model (Sonnet or better)
  - [ ] Prompt: "Is this a generalizable skill? Write a SKILL.md if yes."
  - [ ] Validate output against AgentSkills SKILL.md spec

- [ ] **M2.3: Skill staging**
  - [ ] Write candidate SKILL.md to `~/.openclaw/skills/candidates/`
  - [ ] Notify user: "New skill candidate: X — review and promote to active?"
  - [ ] Promote on user confirmation to workspace skills folder

---

## Phase 3: Learning Hooks — v2 (2026-Q3)

Add online learning on top of the v1 formula. Requires v1 to be running and collecting data.

- [ ] Feedback signal design: what constitutes a "good" response?
- [ ] Log decision + outcome to evaluation store (FR-07)
- [ ] OPE/DR evaluation harness: did retrieval policy improve outcomes?
- [ ] Update formula weights based on evaluation results
- [ ] Canary rollout for policy changes

---

## Phase 4: Scale — v2+ (TBD)

Only if/when single-user SQLite limits are hit.

- [ ] Migrate to Postgres + pgvector (ADR-003 revisit)
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
