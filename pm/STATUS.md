# Project Status: USME — Utility-Shaped Memory Ecology

**Current Stage:** DESIGN
**Last Updated:** 2026-04-04
**Owner:** Alex

---

## Stage Definitions

| Stage | What happens | Exit criteria |
|---|---|---|
| IDEATION | Goal clarification, feasibility, scope boundaries | Problem statement agreed |
| DISCOVERY | Deep questions answered, PRD written, team chosen | PRD approved, open questions resolved |
| DESIGN | Architecture written, UX spec if applicable | architecture.md approved |
| BUILD | Swarm running, code being written | All files exist, builds without errors |
| REVIEW | Code reviewer + security auditor running | review.md complete, critical issues fixed |
| VERIFY | Tests written and passing | All tests green |
| SHIP | Docs written, committed, PR created if applicable | Committed, PR open or merged |
| DONE | Project complete | — |

---

## Stage Log

| Date | Stage | Notes |
|---|---|---|
| 2026-03-30 | IDEATION | Project initiated from USME spec PDF |
| 2026-04-01 | DISCOVERY | PM docs written from spec + transcript analysis. Gap analysis complete. Key decisions resolved. |
| 2026-04-04 | DESIGN | ARCHITECTURE.md complete. Full system design across three planes, shadow mode infra, mode system, ContextEngine interface locked. 27 open decisions documented. Ready for swarm. |

---

## Current Blockers

None. Architecture is ready for swarm launch pending Alex's approval.

---

## Current Stage: DESIGN ✅ — Architecture complete, swarm composition pending

**ARCHITECTURE.md status:** Complete and reviewed. Key additions in final session:
- §3.5 — Three context assembly modes (`psycho-genius`, `brilliant`, `smart-efficient`)
- §6 — Current session history strategy (modes A/B/C, token budget arithmetic)
- §6.3 — Compaction redefined (USME doesn't accumulate; `/compact` = on-demand episode flush)
- §7.1 — ContextEngine interface contract locked from source (`types.d.ts`)
- §7.2–7.3 — Shadow mode as hard v1 requirement with full evaluation infrastructure spec

**Open decisions:** 27 documented in ARCHITECTURE.md §10. Several require algorithm expertise and competitive research.

---

## Swarm Composition

**LOCKED decisions that bound the swarm:** 9 ADRs in DECISIONS.md, plus the ContextEngine interface contract. Swarm evaluates open decisions and implements; it does not revisit locked decisions without flagging.

**Proposed agent roster:**

| Agent | Role | Key responsibilities |
|---|---|---|
| Architect | System architect | Schema finalisation, interface boundary validation, open decision adjudication, coordinates others |
| Algo Expert | Algorithm designer | Selection formula tuning (§3.3), per-mode parameter design (§3.5), scoring thresholds, recall depth strategy. This role requires deep retrieval/ranking expertise — not just implementation |
| Competitive Intel | Research agent | Survey Mem0, LangMem, Zep, MemGPT/Cognee, RAG retrieval systems for algorithm patterns. Use patterns as inspiration; do not reuse or copy code. Produce a briefing doc before Algo Expert designs |
| Coder | Implementation | Core library (`usme-core`): hot path, extraction pipeline, nightly job, ContextEngine adapter |
| Tester | QA | Unit + integration tests, shadow mode harness validation, latency benchmarks |
| Prompt Engineer | LLM prompt design | Haiku/Flash extraction prompts, Sonnet merge/consolidation prompts, skill SKILL.md drafting prompt. Adapts Mem0 few-shot patterns; designs independently |

**Sequencing:** Competitive Intel runs first (or in parallel with Architect) and delivers a briefing. Algo Expert uses the briefing to design mode parameters and formula weights. Coder implements from Algo Expert's spec. Tester runs alongside Coder. Prompt Engineer can run in parallel with Coder once extraction interface is stable.

**Why this roster:**
- The algorithm quality is the most uncertain and highest-leverage part of the build. A dedicated Algo Expert with a competitive intel briefing is the best way to get good numbers for mode thresholds, formula weights, and recall depths.
- Without the Prompt Engineer, the extraction prompts will be mediocre. The few-shot examples are the most important prompt engineering work in the project.
- Shadow mode infrastructure is non-trivial — Tester must treat it as a first-class deliverable, not an afterthought.

---

## Next Actions

1. Alex reviews and approves swarm composition above
2. Rufus reads ruflo-swarm skill, writes the swarm prompt
3. Create standalone git repo (`usme-claw`) with monorepo structure
4. Launch swarm
5. Monitor: Competitive Intel + Architect complete first → Algo Expert designs → Coder + Tester + Prompt Engineer build
