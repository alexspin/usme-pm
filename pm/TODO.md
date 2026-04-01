# TODO: USME

**Last updated:** 2026-04-01

Priority tiers: P0 = must / blocking · P1 = should / this cycle · P2 = nice to have

---

## P0 — Discovery Blockers (must answer before DESIGN)

- [x] Answer Q1: Pain driver = memory not prioritized + memory doesn't compound into skills (@Alex — 2026-04-01)
- [x] Answer Q2: Skill pass rate + memory priority audits + context token savings (@Alex — 2026-04-01)
- [x] Answer Q3: Ports and adapters — usme-core (portable) + openclaw adapter in rufus-plugin (@Alex — 2026-04-01)
- [x] Answer Q4: Shadow mode first — parallel to LCM, hard cutover after validation (@Alex — 2026-04-01)
- [x] Answer Q5: sqlite-vec + node:sqlite built-in — pure C, zero native build, in-process (@Rufus — 2026-04-01)
- [x] Answer Q6: Match LCM latency — synchronous in-process hot path, no remote calls, measure baseline first (@Alex — 2026-04-01)

---

## P1 — Design (once discovery is resolved)

- [ ] Write architecture.md — storage schema, hot path flow diagram, async path flow
- [ ] PRD review — Alex approves draft PRD before moving to DESIGN
- [ ] Spike: validate node:sqlite + sqlite-vec loadExtension() works end-to-end on Node v25.8.1 (quick smoke test before schema design)
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
