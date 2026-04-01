# TODO: USME

**Last updated:** 2026-04-01

Priority tiers: P0 = must / blocking · P1 = should / this cycle · P2 = nice to have

---

## P0 — Discovery Blockers (must answer before DESIGN)

- [ ] Answer Q1: What is the actual pain driver? (missing context? token bloat? pitch?) (@Alex)
- [ ] Answer Q2: Define success signals and measurement approach (@Alex + Rufus)
- [ ] Answer Q3: rufus-plugin vs standalone plugin? (@Alex)
- [ ] Answer Q4: Shadow mode rollout vs hard cutover strategy? (@Alex + Rufus)
- [ ] Answer Q5: Select SQLite vector extension — evaluate sqlite-vec vs alternatives (@Rufus)
- [ ] Answer Q6: Agree on hot path latency budget (@Alex)

---

## P1 — Design (once discovery is resolved)

- [ ] Write architecture.md — storage schema, hot path flow diagram, async path flow
- [ ] PRD review — Alex approves draft PRD before moving to DESIGN
- [ ] Prototype sqlite-vec: verify it works on Node v25 (no native build issues like better-sqlite3)
- [ ] Define memory item ingest rules: what gets stored from each turn? (tool calls, messages, only certain types?)
- [ ] Define provenance tiers and scoring weights (formula spec for ADR-004)
- [ ] Design the shadow mode evaluation log format

---

## P2 — Backlog

- [ ] Write AGENT_ROSTER.md for the USME build swarm (who builds what)
- [ ] Benchmark: measure current LCM token footprint per turn (baseline for success metric)
- [ ] Research: sqlite-vec HNSW index performance on single-user workload at scale (1K, 10K, 100K memories)
- [ ] Design contradiction detection algorithm for memory critic (how do we detect M2 supersedes M1?)
- [ ] Draft skill distillation prompt template (what does USME send to Sonnet to generate a SKILL.md?)

---

## Done

- [x] USME spec PDF authored — 2026-03-30
- [x] Gap analysis vs LCM completed — 2026-03-30
- [x] Key decisions logged (ADR-001 through ADR-005) — 2026-04-01
- [x] PM docs initialized (STATUS, PRD, DECISIONS, OPEN_QUESTIONS, TENETS, ROADMAP, TODO) — 2026-04-01
