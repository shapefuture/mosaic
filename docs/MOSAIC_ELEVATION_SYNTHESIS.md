# Elevating a codebase to "MOSAIC level": analysis and synthesis

## Scope and method

- Input: `1394908f-mosaic.html` (MOSAIC Ω landing page) and the 8 zip archives in the repository root. The landing page was supplied as an upload and is **not committed to this repository**. Sections that cite it (§1, §4C/4D/4G, §5) cannot be verified from the repository alone. Commit it, or treat those sections as referencing an out-of-band input.
- Extracted each archive into a scratch directory (not committed), initialised a throwaway git snapshot in each, and indexed it with GitNexus 1.6.12 (`gitnexus analyze --index-only`). Cross-checked concepts with ripgrep 14.1.0.
- Ran the test suites that could be run locally (pytest 9.2.0 in a venv).

### Reproduction

Environment: Python 3.13.18, Linux 6.18.44-fc-v236. Dependencies installed into a venv with `pip install pytest click 'httpx[http2]' pydantic rich cryptography hypothesis pyyaml gcf-python` for chimera, and `pytest-cov pytest-xdist pytest-testmon coverage` for the declared dev extras. Archives extracted to a scratch directory, not into the repository.

```bash
# mosaic-omega-7.0.0 (extracted to <scratch>/zips/mosaic-omega-7.0.0/source)
cd <scratch>/zips/mosaic-omega-7.0.0/source
PYTHONPATH=src python -m pytest -q -p no:cacheprovider tests        # 145 passed

# chimera_code_latest_20260813 (extracted to <scratch>/zips/chimera_code_latest_20260813/chimera)
cd <scratch>/zips/chimera_code_latest_20260813/chimera
PATH=<venv>/bin:$PATH PYTHONPATH=. python -m pytest -q -p no:cacheprovider tests
# 758 passed, 1 failed, 12 skipped with dev extras (the 1 failure is the zombie-process case in §7)
```

Without the dev extras and with the venv not on PATH, chimera gives 748 passed, 11 failed, 12 skipped. This earlier result is the one reported in §2 and §3.

- Not run: network-dependent benchmarks, formal backends (Lean, Dafny, Verus, Z3, Frama-C), Docker/bubblewrap sandboxes, the self-reported gauntlet JSONs (later re-run in §7).

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
| `chimera_code_latest_20260813.zip` | Python `chimera-agent-harness` 4.4.0: "evidence-gated, transactional harness for coding agents"; Rust graph mirror; Tauri GUI | ~59k LOC `.py`, 278 test files | 993 files · 14,310 symbols · 29,304 edges | **758 pass, 1 fail, 12 skip** with declared dev extras (11 fail without them; see §7) |
| `event-driven-ai-operating-system.zip` (+ `(1)`) | Next.js 16 + Postgres/Drizzle "AOS v3" control plane: hibernating agent identities, HMAC-signed capability envelopes, idempotent receipts | 21 / 43 files | 21 / 43 files · 291 / 759 symbols | not run |
| `serverless-ai-agent-architecture.zip` (+ `(1)`, `(2)`) | Next.js/Drizzle agent-architecture template variants | 53 / 22 / 57 files | 1,111 / 204 / 1,192 symbols | not run |
| `workspace-01a1acec-….zip` | Research notes: catalogue of ~250 AI code-review tools | Markdown only | n/a | n/a |

Two observations about the archives themselves:

- `mosaic-omega` has version drift, checked against the extracted files: the archive is named `mosaic-omega-7.0.0.zip`, `source/pyproject.toml` says `version = "7.0.0"`, `source/VERSION` says `6.0.1`, and the only source manifest in `source/` is `source/SOURCE_MANIFEST_5.0.0.json`. A single version source would remove this.
- `chimera` contains no "mosaic" naming in its original archive. Its concepts overlap (evidence gating, verification, sandboxing, archiving unused surfaces), but it is a separately named project at a different version. Whether it is a successor to MOSAIC is a question for the owner; the files alone do not establish it.

## 3. Test results (first pass; chimera superseded by §7)

The chimera promotion-gate test listed below passes once the declared dev extras are installed. §7 has the current root-cause analysis.

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
Action: order checks cheap → deep, stop at the first decisive failure, record cost per stage, and rerun only the dependency cone of a change. Verify the promotion gate on real inputs first (see §7).

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
2. **Gate**: promotion = cascade pass + counterexample replay + local re-verification. Verify the promotion gate on real inputs first (see §7).
3. **Memory and routing**: store successes, failures and counterexamples; look up before generating (4C, 4D).
4. **Ledger**: record cost, latency, CRR and OAR per route. Do this before building any compilation tier.
5. **Compile tier**: only where the ledger shows sustained reuse (4G).
6. **Sharing**: export packages with provenance; importers re-verify locally (4F).
7. **Hygiene**: single version source, archive unused surfaces, label all metrics (4H).

Success criteria, all measurable from the ledger: CRR rising over time, frontier-call fraction falling for covered workload families, zero regressions on replay, and no promotion without a passing evidence record.

## 6. Caveats

- No network benchmarks were run; no claim is made that any archive beats a frontier model.
- The landing page's numbers (18,420 skills, costs, percentages) are illustrative by its own labelling.
- The mosaic-omega gauntlet was re-run in §7 and is self-validating; it does not evidence the claims it reports.
- GitNexus graphs were used for stats and as an index; the findings above rely on ripgrep counts and direct reading, not on graph queries.

## 7. Deeper functional analysis (supersedes §2–§4 where they differ)

Method: AST inventory of every Python package (LOC, functions, classes, docstrings); line coverage measured from each suite's own tests; reading of the core logic behind the main claims; for the Next.js projects, route inventory, import scan, and reading of engine, gateway and browser code. Not covered in depth: the remaining Next.js variants' UIs, every chimera module, and the research notes beyond their structure.

### mosaic-omega 7.0.0 (`mosaic_omega`)

- 174 modules, 12,330 LOC, 663 functions, 286 classes. 145 tests pass. Line coverage 84% of 8,657 statements; 166 of 174 modules at or above 40%.
- **Substantive and exercised:** HMAC attestation (`security/provenance.py`; a symmetric key, not hardware attestation); a deny-by-default policy check (`security/policy.py`); a greedy set-cover verifier portfolio (`verification/portfolio.py`); n-gram procedure mining with support counts, causal scoring and delta-debugging (`learning/procedures.py`); an invalidation graph that recomputes only downstream verifiers (`compiler/residual.py`); a 30-subcommand CLI (`cli/main.py`).
- **Zero coverage:** `attestation/providers.py`, `isolation/firecracker.py`, `isolation/process.py`, `profiling/resources.py`, `runtime/economics.py`. Under 20%: `data/remote_trajectories.py`, `ir/codegen.py`. The Firecracker and cloud backends are thin adapters that require host tools.
- **The omega7 gauntlet is self-validating.** `gauntlets/omega7.py` runs on hand-built inputs and then asserts results about them:
  - residual savings = `1 − cost / 9.3`, where 9.3 is a hard-coded baseline;
  - the promotion gate is `PromotionGate(True, True, True, True, True)`, so the self-hosting check is a constant;
  - `gpu_work_fraction` and `frontier_call_fraction` are constants `0.0`, so their scores are always 1.0;
  - hotness inputs are fixed (100,000 uses, 0.999 success), so the `simd` tier follows by construction;
  - formal backends detected on this host: 0. The file's own comment says they are unavailable here.
  
  Its `all_checks_passed: true` therefore shows that the wiring works. It does not show the claimed savings, GPU avoidance or frontier avoidance.
- Version drift: archive 7.0.0, `VERSION` file 6.0.1, source manifest 5.0.0.

### chimera 4.4.0 (flat package `chimera/`)

- 364 modules in `chimera/`, 59,156 LOC including subpackages, 2,869 functions, 1,146 classes. 278 test files.
- **Test results:** with the declared dev extras (`pytest-xdist`, `pytest-testmon`, `pytest-cov`) installed and the venv on PATH: 758 pass, 1 fail, 12 skip.
- **The 11 original failures were mostly environmental.** Harness tests spawned a bare `python` without pytest; acceleration and evidence-boundary tests needed `pytest-xdist` and `pytest-testmon`. Installing the declared extras fixed them. I did not confirm each cause individually.
- **The remaining failure is a deliberate conservative verdict plus a container artifact.** `test_managed_step_deadline_terminates_its_process_group` expects `cancelled`. The code returns `unknown` because `process_group_absent` uses `killpg(pgid, 0)`, which still succeeds for a zombie. After the test, two zombies remain, reparented to PID 1 (`process_api`), which here does not reap children. On a host whose init reaps children this should pass; I did not verify that.
- **Coverage:** 79% of 35,273 statements. `adapters/` is 0%. `orchestrator.py` is 20% and `dbos_durable_workflow.py` is 26%.
- **Substantive modules:** hash-chained WAL (`wal.py`); transactional workspace (`branch.py`); process-group cancellation (`process_control.py`); WASM sandbox with fuel and memory limits (`sandbox.py`, using `wasmtime` if installed); bubblewrap certification; effect-typed action IR (`action_ir.py`); LTL-monitored verification (`verifier.py`); MCP client, server and proxy; signed skill packages with rollback (`skills.py`); signed release provenance with SBOM (`release_provenance.py`); tenant authorization.
- **Optional backends** are guarded imports: `wasmtime`, `dbos`, `restate`, `e2b`, `hyperlight`, `granian`, `duckdb`, `opentelemetry`, `tree_sitter`. The docstrings in `graph.py` mention Kuzu, but no Kuzu import exists. The code graph does not match its docstring.
- **CLI:** about 150 commands including nested groups. A large share of `platform` is planning, audit and report generation (for example `gate-dashboard`, `capability-per-kb-plan`, `sota-invariant-audit`). These describe readiness; they do not add capability.

### Next.js projects (five archives)

- **Common stack:** Next 16, React 19, Drizzle, Postgres (`pg`). No import of Cloudflare, Pi, Playwright, Stagehand, Cedar, pgvector, R2, Firecracker or any model SDK. None has a lockfile.
- **`event-driven-ai-operating-system`** (14 files, 2,755 lines): 8 REST routes. HMAC authority tokens are real crypto. `executeDeterministicTask` sleeps for a fixed latency and returns synthetic output. The "blind critic" is an arithmetic score over string lengths. The TRIZ resolver maps keywords to fixed templates.
- **`event-driven-ai-operating-system__1_`** (35 files, 6,410 lines): not analysed beyond inventory and the README claim "a working fullstack implementation."
- **`serverless-ai-agent-architecture`** (v1, 46 files, 4,820 lines): the model gateway makes a real OpenAI-compatible call when a key is set and otherwise degrades to a deterministic fallback. The HTTP tool blocks private IPs and keeps cookie jars. The `node:vm` Tier-1 sandbox says in its own comment that it is not a hostile-code boundary. Not confirmed: whether `browser.ts` drives a real browser.
- **`serverless-ai-agent-architecture__2_`** (50 files, 10,818 lines, 16 routes): `gatewayCall` never calls a provider, and the engine contains no `fetch`. Browser contexts are database rows. The scheduler simulates platform eviction. Type-checks with `tsc --noEmit` after `npm install` (390 packages).
- **`serverless-ai-agent-architecture__1_`** (15 files): not analysed beyond inventory (fan-out, snapshot and simulation routes).

### Workspace archive

- `research/notes.md` (281 lines): a dated raw log of a late-2026 search sweep with URLs.
- Two Markdown catalogues of AI code-review tools (1,286 and 481 lines). These are reference prose, not code, and I have not verified their claims.

### What changes in the recommendations

1. The biggest risk in this set is the gap between reported and measured capability. The mosaic-omega gauntlet and the Next.js engines both report results that their code produces by construction. This strengthens recommendation H (honest evidence boundary) and suggests making the omega7 gauntlet compute its inputs from real runs before any of its numbers are cited.
2. The promotion-gate test is not a real defect. Its inputs and the environment need checking, and the gate needs real evidence rather than constant `True` arguments.
3. chimera is the strongest engineering base in this set: real backends, guarded optional dependencies, and a conservative cancellation model. Its main gaps are the 0% `adapters/` package and the planning-heavy CLI.
