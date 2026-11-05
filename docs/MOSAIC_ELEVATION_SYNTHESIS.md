# Elevating a codebase to "MOSAIC level": analysis and synthesis

## Scope and method

- Input: `1394908f-mosaic.html` (MOSAIC Ω landing page) and the 8 zip archives in the repository root.
- Extracted each archive into a scratch directory (not committed), initialised a throwaway git snapshot in each, and indexed it with GitNexus 1.6.12 (`gitnexus analyze --index-only`). Cross-checked concepts with ripgrep 14.1.0.
- Ran the test suites that could be run locally (pytest 9.2.0 in a venv).
- Not run: network-dependent benchmarks, formal backends (Lean, Dafny, Verus, Z3, Frama-C), Docker/bubblewrap sandboxes, the self-reported gauntlet JSONs.

## 1. What "MOSAIC level" means

The landing page defines the target. Its claims are labelled "illustrative" in the page itself, so I treat the page as a **specification**, not as evidence. The spec, distilled:

1. **Proof-carrying capability**: every reusable unit carries preconditions, transformation, expected effect, verifier, evidence, counterexamples, cost model, applicability model, provenance and lineage.
2. **Cheap-to-deep verification cascade**: syntax → types → compile → unit tests → static analysis → property tests → fuzzing → differential execution → symbolic reasoning → formal proof → model arbitration last. Only survivors become capabilities.
3. **Retrieval-first control law**: eliminate → exact capability → generalized capability → deterministic program → specialist → remote model → frontier. Spend only while marginal value exceeds cost.
4. **Counterexamples are first-class**: failures are stored, used to mutate, and re-verified (v11 → v12 loop).
5. **Cost decay through compilation**: verified reuse moves work down a ladder (interpreted → compiled → native/SIMD → specialised operator).
6. **Collective reuse gated by reproducibility**, not reputation; no raw source leaves the machine.
7. **Honest evidence boundary**: state what ran, what was not available, and what is unproven.

## 2. Inventory of the archives

| Archive | What it is | Size | GitNexus index | Verified locally |
|---|---|---|---|---|
| `mosaic-omega-7.0.0.zip` | Python "MOSAIC Ω" computational compiler/runtime, `mosaic` CLI | 174 `.py` files, ~12.3k LOC, 80 test files | 546 files · 6,207 symbols · 9,831 edges | **145 tests pass** |
| `chimera_code_latest_20260813.zip` | Python `chimera-agent-harness` 4.4.0: "evidence-gated, transactional harness for coding agents"; Rust graph mirror; Tauri GUI | ~59k LOC `.py`, 278 test files | 993 files · 14,310 symbols · 29,304 edges | **748 pass, 11 fail, 12 skip** |
| `event-driven-ai-operating-system.zip` (+ `(1)`) | Next.js 16 + Postgres/Drizzle "AOS v3" control plane: hibernating agent identities, HMAC-signed capability envelopes, idempotent receipts | 21 / 43 files | 21 / 43 files · 291 / 759 symbols | not run |
| `serverless-ai-agent-architecture.zip` (+ `(1)`, `(2)`) | Next.js/Drizzle agent-architecture template variants | 53 / 22 / 57 files | 1,111 / 204 / 1,192 symbols | not run |
| `workspace-01a1acec-….zip` | Research notes: catalogue of ~250 AI code-review tools | Markdown only | n/a | n/a |

Two observations about the archives themselves:

- `mosaic-omega` has version drift: the archive is named 7.0.0, `VERSION` says `6.0.1`, and the source manifest says `5.0.0`. A single version source would remove this.
- `chimera` contains no "mosaic" naming in its original archive. Its concepts overlap (evidence gating, verification, sandboxing, archiving unused surfaces), but it is a separately named project at a different version. Whether it is a successor to MOSAIC is a question for the owner; the files alone do not establish it.

## 3. Test results (what was actually reproduced)

- **mosaic-omega**: all 145 tests pass. The gauntlet JSON (`residual_savings_ratio` 0.989, `all_checks_passed: true`) was **not** re-run; treat it as self-reported.
- **chimera**: 11 failures, all in test-harness or environment-sensitive areas:
  - `test_test_acceleration.py` and `test_evidence_boundary_regressions.py`: one failure explicitly reports `pytest-testmon is unavailable`.
  - `test_dynamic_coverage_collector.py` (×2) and `test_dynamic_coverage_context_escaping.py`: coverage-collector mapping.
  - `test_autonomous_control_plane.py`: managed-step deadline process-group termination.
  - `test_candidate_promotion_gate.py`: the promotion gate itself does not allow the clean authorised candidate. This matters most, because it is the core gate.
  - `test_harness_improvements.py` (×2): sandboxed test materialisation and focused coder patch.

  I did not diagnose root causes. Several likely depend on missing host tools (testmon, bubblewrap, SCIP), but the promotion-gate failure needs a look regardless.

## 4. Ways to elevate a codebase to MOSAIC level

Ranked by leverage, each tied to what the archives already contain.

**A. Proof-carrying capability record (highest leverage).**
Evidence: `mosaic-omega` `ComputationalUnit` / `ProgramIR` with explicit effects, budgets, contracts, dependencies and provenance; `capability/` content-addressed package; chimera's evidence receipts.
Action: define one schema for every promoted change: preconditions, transformation reference, verifier references, evidence hash, counterexample list, cost, lineage (parent version), provenance. Nothing is "learned" without this record.

**B. Promotion gate as a verification cascade with cost accounting.**
Evidence: `mosaic-omega` verifier portfolio (choose the smallest evidence set meeting a confidence/cost bound) and residual verification graph (only invalidated obligations rerun); chimera's `candidate_promotion_gate` and coverage collector.
Action: order checks cheap → deep, stop at the first decisive failure, record cost per stage, and rerun only the dependency cone of a change. Fix the failing promotion-gate test first.

**C. Counterexamples as data, with a mutate-and-revalidate loop.**
Evidence: the page's v11 → v12 loop. In code, counterexample handling is thin: `counterexample` appears in only 14 `.py` hits in `mosaic-omega` and 13 in chimera.
Action: persist each failure with its failure condition; use stored counterexamples as regression inputs; retire a capability only when a counterexample is reproducible.

**D. Retrieval-first routing with a cost ledger.**
Evidence: `mosaic-omega` control/strategy modules, chimera `route_pipeline.py`, and the page's control law.
Action: before any generation, look up an exact capability, then a generalised one, then deterministic code, then a model. Record per-route cost and latency. Track two numbers: CRR = reused / total, and OAR = 1 − C_oracle / C_baseline (both defined on the page).

**E. Deny-by-default effects and sandboxing.**
Evidence: `mosaic-omega` effect system enforced before execution; chimera's bubblewrap probes, controlled egress and admission modules; EDA's signed capability envelopes.
Action: every capability declares its effects; execution is denied unless granted; sandbox egress is an explicit, tested policy.

**F. Reproducibility-based sharing instead of reputation.**
Evidence: EDA's idempotent receipts and HMAC-signed envelopes; `mosaic-omega` federation/ package shape.
Action: an imported capability is re-verified locally before use; trust comes from re-running its verifier, not from its origin.

**G. Compile hot paths only after reuse is measured.**
Evidence: `mosaic-omega` hotness compiler (interpreted → WASM → native → SIMD). The page's ladder assumes reuse exists.
Action: gate compilation on a measured reuse threshold in the ledger. Without that data, compilation is premature.

**H. Honest evidence boundary and hygiene.**
Evidence: chimera's `archived_experimental/` with receipts ("prototype names cannot be mistaken for supported capabilities"); the page's own "illustrative" labels.
Action: single version source; archive unused surfaces with a receipt; label every metric as measured, reproduced, self-reported or illustrative.

**I. Agent-as-identity runtime (lower priority).**
Evidence: EDA/serverless: an agent is an identity plus role, budget and persistent knowledge; work wakes a pool, commits a receipt, sleeps.
Action: useful as the runtime tier, not as the core learning loop.

**J. Tool catalogue as verifier input.**
Evidence: the research archive lists ~250 AI code-review tools with URLs and dated errata.
Action: use it to choose verifiers for the portfolio (B), keeping the dates in mind.

## 5. Synthesis: best of all

Use each archive for the part it does best:

- **Host and gating discipline**: chimera (transactional harness, promotion gate, sandbox, archive of unused surfaces).
- **Data model**: mosaic-omega (`ComputationalUnit`, effects, verifier portfolio, capability package, residual verification).
- **Collective exchange**: EDA (signed envelopes, idempotent receipts) with reproducibility-based trust.
- **Verification policy**: the page's cheap-to-deep cascade, with cost accounting.
- **Counterexample memory**: first-class, reused as regression input.

Roadmap:

1. **Schema**: one capability record (section 4A) for every promoted change.
2. **Gate**: promotion = cascade pass + counterexample replay + local re-verification. Fix the failing promotion-gate test first.
3. **Memory and routing**: store successes, failures and counterexamples; look up before generating (4C, 4D).
4. **Ledger**: record cost, latency, CRR and OAR per route. Do this before building any compilation tier.
5. **Compile tier**: only where the ledger shows sustained reuse (4G).
6. **Sharing**: export packages with provenance; importers re-verify locally (4F).
7. **Hygiene**: single version source, archive unused surfaces, label all metrics (4H).

Success criteria, all measurable from the ledger: CRR rising over time, frontier-call fraction falling for covered workload families, zero regressions on replay, and no promotion without a passing evidence record.

## 6. Caveats

- No network benchmarks were run; no claim is made that any archive beats a frontier model.
- The landing page's numbers (18,420 skills, costs, percentages) are illustrative by its own labelling.
- The mosaic-omega gauntlet JSON was not re-run; chimera's 11 failures were not root-caused.
- GitNexus graphs were used for stats and as an index; the findings above rely on ripgrep counts and direct reading, not on graph queries.
