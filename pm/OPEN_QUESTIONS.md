# Open Questions: USME

Questions that must be answered before moving to DESIGN.
When resolved, move to DECISIONS.md and mark `resolved` here.

| # | Question | Owner | Raised | Status |
|---|---|---|---|---|
| Q1 | What is the actual pain driver? | Alex | 2026-03-30 | open |
| Q2 | How do we define and measure success? | Alex + Rufus | 2026-03-30 | open |
| Q3 | Ship as rufus-plugin or new standalone plugin? | Alex | 2026-04-01 | open |
| Q4 | Transition strategy — shadow mode first or hard cutover? | Rufus | 2026-04-01 | open |
| Q5 | Which SQLite vector extension? | Rufus | 2026-04-01 | open |
| Q6 | Latency budget for hot path packaging? | Alex | 2026-04-01 | open |

---

## Q1 — What is the actual pain driver?

**Question:** What is driving USME? Is it:
- (a) Rufus missing context it should have — wrong answers because relevant history wasn't in context?
- (b) Token cost / bloat — too many tokens per turn, slow responses?
- (c) Proving out the "organizational memory that compounds" pitch for the Rufus product?
- (d) All of the above, but with a primary driver?

**Why it matters:** The answer changes v1 scope significantly.
- If (a): prioritize retrieval quality (FR-03) and multi-mode packaging (FR-04)
- If (b): prioritize tighter selection + token budget enforcement
- If (c): prioritize skill distillation (FR-08) since that's the most pitchable capability

---

## Q2 — How do we define and measure success?

**Question:** What signals tell us USME is working better than lossless-claw? Options:
- Alex explicitly rates response quality (thumbs up/down logged to JSONL)
- Proxy metrics only: token count per turn, compression ratio, retrieval latency
- An A/B harness that runs both context engines on the same prompts and compares outputs
- LLM-as-judge evaluation against a held-out test set

**Why it matters:** Without a success metric, we can't know if the learned selection policy is improving.
FR-07 (online learning hooks) requires feedback signals — this decision defines what those signals are.

---

## Q3 — Ship as rufus-plugin or new standalone plugin?

**Question:** USME is a replacement context engine. Should it live:
- (a) Inside rufus-plugin (as a new module alongside context-logger, dashboard, remote-exec)
- (b) As a new standalone plugin (e.g., `usme-claw`) that claims the contextEngine slot independently
- (c) As a fork/extension of lossless-claw itself

**Why it matters:** Determines repo structure, deployment complexity, and the upgrade path for users who might want USME without the full rufus-plugin stack.

**Rufus's lean:** (a) keeps ops simple for now; (b) makes more sense if USME is eventually a publishable, standalone product. This is a scope question for Alex.

---

## Q4 — Transition strategy — shadow mode first or hard cutover?

**Question:** lossless-claw is working today. When USME is ready, how do we switch?
- (a) Feature flag: USME runs in shadow mode (same LCM context goes to model), logs what it *would have* sent — compare quality before enabling
- (b) Hard cutover: disable lossless-claw, enable USME, monitor closely
- (c) Gradual: USME handles new sessions, LCM handles existing until sessions cycle

**Why it matters:** Shadow mode first is lower risk but requires building evaluation tooling. Hard cutover is faster but blind.

**Rufus's lean:** (a) shadow mode first — we have the JSONL logging infrastructure from the distiller work, extend it for USME.

---

## Q5 — Which SQLite vector extension?

**Question:** Per ADR-003, we use SQLite + vector extension. Candidates:
- [`sqlite-vec`](https://github.com/asg017/sqlite-vec) — Alex Garcia's extension, MIT license, WASM-compatible, actively maintained
- [`sqlite-vss`](https://github.com/asg017/sqlite-vss) — same author, older, uses Faiss under the hood, more complex build
- [`vectorlite`](https://github.com/1yefuwang1/vectorlite) — HNSWLib-based, virtual table interface

**Why it matters:** This is the core infrastructure decision. Must pick before starting the data model implementation.

**Rufus's lean:** `sqlite-vec` — WASM-compatible (avoids native compilation issues we hit with better-sqlite3), simpler API, same author as sqlite-vss but more modern.

---

## Q6 — Latency budget for hot path packaging?

**Question:** The spec says P95 packaging latency should be "configurable (50–150ms local-only, 150–600ms with remote refiner)." What's our actual target?

- If we use a fast refiner (Flash/Haiku) in the hot path, expect 150–500ms additional latency
- If we skip the refiner in v1, pure local retrieval + scoring should be ≤50ms for SQLite
- User-visible: every message goes through this path — 200ms+ is perceptible

**Why it matters:** Determines whether we include a fast refiner (Flash/Haiku) in v1's hot path, or defer that to v2.

---

## Resolved

| # | Question | Answer | Resolved |
|---|---|---|---|
| — | — | — | — |
