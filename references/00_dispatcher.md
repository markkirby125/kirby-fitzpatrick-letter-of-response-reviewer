# Letter Of Response Reviewer — Technical Operational Dispatcher

**Framework Author**: William Fitzpatrick (*Writer Science*)  
**Source Lecture**: [I coached 25 writers to learn THIS](https://www.youtube.com/watch?v=gaqIcDkTakA)  
**Parent Collection**: [Master Collection](../../kirby-fitzpatrick-writers-collection/SKILL.md) | [Global Help](../../../kirby-help/SKILL.md)  

---

## 1. Cognitive Foundation: The Review Thread as a Negotiation of Evidence

A code review is not a conversation. It is a **negotiation of evidence conducted in writing, in public, under time pressure, by parties with asymmetric context**. The author holds the mechanism; the reviewer holds the risk. Neither can see the other's model of the system. What closes the gap is not seniority, tone, or insistence — it is an artifact both parties can open.

Most authors answer review with **intent** — *"this is faster"*, *"that's the existing pattern"*, *"I already explained this"* — while the reviewer asked for **mechanism**. That mismatch produces the classic thread: five comments, three rounds, zero resolutions, and a merge that happens for social reasons. Six months later nobody can reconstruct why the loop stayed scoped per row, because the reason lived only in a thread that got collapsed.

**The Letter of Response standard** replaces the rebuttal with a durable, structured artifact. Every review item the author answers moves through exactly three moves:

1. **Acknowledge** — restate the reviewer's concern in the author's words before evaluating it. Acknowledgement is comprehension evidence, not politeness.
2. **Cite concrete change** — for anything accepted, point at the diff at a named commit: `path:line`, plus the test that now covers the behavior. A change described in prose is not a change; it is a claim.
3. **Present benchmark data for disagreements** — a declined concern is settled by measurement with a named baseline, workload, runner, sample size, and the raw command — never by "I don't think so", "we've always done it that way", or "per your request".

Three load-bearing definitions:

- **The Letter of Response (LoR)** — the written set of dispositions covering every review item, patched back into the durable artifact (PR body, ADR, thread comment on the artifact). It is the review's *output*; the diff is only its mechanism.
- **Disposition** — the terminal state of one review item. There are exactly four: `ACCEPTED`, `DECLINED`, `DEFERRED`, `INSUFFICIENT`. **An item in no terminal state is open forever, no matter how many approvals the PR carries.** Approval closes a PR; only dispositions close a thread.
- **Evidence parity** — whoever asserts a delta (latency, correctness, safety, cost) must supply the artifact that would falsify it. The reviewer who says "this will be slow" owes a benchmark shape too. Parity is what keeps the exchange from becoming an authority contest.

Why this binds harder in software than in prose: **engineering artifacts are read by future strangers**. Review comments are the only place where the *why* of a line survives — and they survive only if someone converted discussion into disposition. The lecture pairs this skill with the **Silent Author Test** (same source): that skill governs whether an artifact stands without its author; this one governs the dialogue that produces it.

**The Disagreement Ladder.** Every review disagreement resolves at one of three rungs — and the ladder is strictly ordered:

```text
Rung 0  Silence / dismissal         -> loses by default; the item stays open permanently
Rung 1  Assertion + authority       -> loses to deadlines ("let's watch it in prod")
Rung 2  Benchmark + mechanism       -> wins, and generalizes to the reviewer's workload
```

Rung 1 is the most expensive rung, because it *appears* to end the exchange while leaving the concern unresolved — so it returns later as an incident, an incident review, or a revert.

```text
[ANTI-PATTERN: Assertion Loop]  ->  intent vs. intent; no artifact ever enters
+--- PR #812 — review thread, round 3 ----------------------------------------+
| R: "This will be slow on large tenants."                                     |
| A: "I don't think so."                                                       |
| R: "You're scoping per row."                                                 |
| A: "That's how the old code worked too."                                     |
| R: "..."                                                                     |
+------------------------------------------------------------------------------+
Open items: 5    Terminal dispositions: 0    Rounds: 3    Approvals: 0
Cost: 3 review cycles, 2 side-channel DMs, 1 standup tangent. Six months later
      the reason for per-row scoping is unrecoverable, and the thread reopens.

[LETTER OF RESPONSE]  ->  every item acknowledged, evidenced, and closed
+--- PR #812 — review thread, round 1 (single round) --------------------------+
| R1 per-row scoping        -> ACCEPTED   3a77e02 handler.ts:88 via TenantRepo |
|                                            reconcile_test.go:214 (query cnt) |
| R2 "slow on large tenants"-> DECLINED   bench: p95 480->210 ms, 100k wkld,   |
|                                            baseline 4f9c1ab, 5 runs, +/-6 ms |
| R3 error-message wording  -> DEFERRED   ISSUE-1043, owner: reviewer, 05-02   |
| R4 tenant filter in repo  -> ACCEPTED   same commit as R1                    |
+------------------------------------------------------------------------------+
Open items: 4    Terminal dispositions: 4    Rounds: 1    Approvals: 1
Result: the thread closes on evidence, and the dispositions are patched into the
        PR body so the next reader never has to reopen it.
```

---

## 2. Core Transformation Protocols

1. **Answer every item, in order, by number.** Renumber the reviewer's comments (`R1…Rn`) and respond 1:1. A skipped comment is not a neutral omission — the reviewer reads it as an override, and they will re-raise it next round. No item silently disappears.
2. **Acknowledge before you evaluate.** Open each response with the reviewer's concern restated in your own words. If you cannot restate it precisely, you have not understood it: reply with `INSUFFICIENT` and ask for the specific artifact that would resolve your ambiguity.
3. **Force every item to a terminal disposition.** `ACCEPTED` / `DECLINED` / `DEFERRED` / `INSUFFICIENT` — nothing else is a valid end state. "Good point, will look" is not a disposition; it is a promise with no ledger entry.
4. **Prove acceptance with a diff, never with prose.** Cite the commit and `path:line` at the PR head, and name the test or assertion that now covers the behavior. *"Fixed"* is unfalsifiable; `handler.ts:88` at `3a77e02` is auditable in five seconds.
5. **Disagreements require benchmark data — no exceptions.** A decline carries: claim → mechanism → measurement → baseline ref (SHA/branch/tag) → workload → runner → runs/dispersion → raw command. A number without a baseline is a memory; a number without a command cannot be reproduced.
6. **If no measurement exists, decline is not available — only `INSUFFICIENT` with a named measurement.** This is the single most important rule. Nobody may win an unmeasured disagreement by confidence: either someone produces the profile/benchmark/query count, or the concern stays open with an owner and a date.
7. **Benchmark the reviewer's claim, not your own.** When a reviewer asserts an effect ("this will be slow", "this breaks idempotency"), measure the *asserted* effect. Disagreement answered with an unrelated favorable number reads as evasion.
8. **Bind each measurement to the mechanism that isolates it.** `p95 −56%` invites *"will it hold on my data?"*; `p95 −56% because the N+1 fetch is one batched query, isolated by the query-count assertion at reconcile_test.go:214` answers it in the same sentence.
9. **Concede fast and out loud.** *"You're right — fixed in `3a77e02`."* A concession costs one line; a defended mistake costs a review cycle, a heated thread, and the reviewer's trust in every other claim you make. Conceding is the cheapest credibility purchase available.
10. **Separate mechanism from preference, and label the labels.** `FACT` resolves to an artifact now; `JUDGMENT` names its criterion (the style guide section, the precedent PR, the error-handling convention) and stays labeled. Never launder a preference as a correctness rule; never soften a correctness defect into a preference to end a thread.
11. **Never defer to the reviewer to avoid owning the decision.** *"Per your request"*, *"happy to change it if you feel strongly"*, *"whatever the team prefers"* transfer ownership of a decision nobody then owns. Accept it (and say why it is right), decline it (with data), or defer it (with an issue) — do not launder it.
12. **Patch every resolution back into the durable artifact.** A disposition that exists only in a comment thread is not recorded, it is spent. Fold the LoR into the PR body, the ADR, or a doc change, then collapse the thread.
13. **One round must reduce open items.** Track the count (`open → terminal`). A round that closes nothing is not progress; it is signal to change the axis — escalate to a synchronous call, narrow the disputed surface, or escalate the disagreement's axis (correctness vs. style vs. schedule) explicitly.
14. **Strip apology and unearned authority in equal measure.** *"Sorry, I should have…"*, *"obviously"*, *"clearly"*, *"this is much cleaner"* all carry zero information and read as either anxiety or assertion. State the fact, the diff, or the number. (Related dispatchers: purge zero-information intensifiers with the [Lexical Anti-Bloat Filter](../../kirby-fitzpatrick-lexical-anti-bloat-filter/SKILL.md).)
15. **Close the loop on deferred items before merge.** `DEFERRED` without an issue ID, owner, and date is `ACCEPTED`-shaped avoidance: the concern is real, the work is unowned, and it will surface after release. Filed and linked, or it is not deferred.

### 2.1 The four dispositions and their required evidence

| Disposition | Means | Required evidence | Item is terminal when |
|---|---|---|---|
| `ACCEPTED` | The reviewer is right; code or behavior changed | Commit SHA at PR head + `path:line` + the test/assertion covering it | The commit is pushed and cited in the response |
| `DECLINED` | The concern does not hold as stated | Claim + mechanism + measurement + baseline ref + workload + runner + runs/dispersion + raw command | Evidence is posted in-thread **and** patched into the artifact |
| `DEFERRED` | Valid, but belongs to a different change | Issue/ticket ID + owner + date + one line on sequencing | The issue exists and is linked from the response |
| `INSUFFICIENT` | Cannot decide yet; a specific artifact is missing | The exact artifact requested (profile, query count, failing input, load run), who produces it, and by when | The artifact is attached; the item re-enters as `ACCEPTED`/`DECLINED`/`DEFERRED` |

### 2.2 Anatomy of one response item (the 3-move template)

```markdown
**R2 — "the batched query will be slow on large tenants" (blocking)**
[Move 1: ACKNOWLEDGE] The concern is that batching shifts work from the DB round trip
into a wide `IN` clause, so p95 degrades as tenant size grows.

[Move 3: BENCHMARK DATA — disagreement] DECLINED
- Claim:      p95 480 ms → 210 ms at 100k invoices; no regression up to 1M rows
- Mechanism:  N+1 fetch → one batched query (handler.go:88); query count 1,240 → 41
              isolated by the query-count assertion in reconcile_test.go:214
- Workload:   prod-sample-100k (and prod-sample-1m for the scaling check)
- Baseline:   main @ 4f9c1ab   ·  PR head 3a77e02  ·  runner gha:ubuntu-22.04, 4 vCPU
- Runs:       median of 5, p95 spread ±6 ms
- Command:    make bench-reconcile WORKLOAD=prod-sample-100k RUNS=5
- Reverses if: tenant invoices exceed 1M (untested; would re-open as a new item)

[Move 2: CONCRETE CHANGE — the part of the concern I did accept] ACCEPTED 3a77e02
- Per-row scoping removed: handler.ts:88 now routes through `TenantRepo.findOrders()`
- Covered by: TestReconcileTenantScope (reconcile_test.go:214)
```

### 2.3 Transformation table: response anti-patterns and clean replacements

| Anti-Pattern (as written in-thread) | What the reviewer hears | Clean Replacement |
|---|---|---|
| "Fixed." | Where? Verified how? | "Fixed in `3a77e02` — `handler.ts:88` routes through `TenantRepo`; covered by `TestReconcileTenantScope`." |
| "I disagree." | Authority vs. authority; thread escalates | "Declined with data: p95 480 → 210 ms, `make bench-reconcile WORKLOAD=prod-sample-100k RUNS=5`, baseline `main @ 4f9c1ab`, median of 5." |
| "I already explained this above." | You consider the item closed; it isn't | "Restating: the loop is scoped per tenant (handler.ts:88); query count is 41, asserted at reconcile_test.go:214." |
| "That's just a style preference." | Correctness concern dismissed by label | "This is `JUDGMENT`, criterion = `CONTRIBUTING.md` §error-handling rule 3; the current shape matches it, so declining unless that rule changed." |
| "Out of scope for this PR." | Work is being dropped silently | "Deferred: ISSUE-1043, owner reviewer, due 2026-05-02; sequencing rationale — touches the response cache in PR #830." |
| "Should be fine / works on my machine." | Unfalsifiable verdict | "14 unit cases + CI run `build/4471` green on PR head; not covered: >10k-ID batches (open, owner author, before merge)." |
| "Happy to change it if you feel strongly." | Decision laundered onto the reviewer | Accept it and say why it is right, or decline with data. Ownership never transfers. |
| "This is the standard pattern." | Tribal knowledge; no referent | "Used by the other 7 call sites in `billing/*.ts`; extracted as `RetryPolicy` so the 8th cannot diverge." |
| "Sorry, I should have caught that." | Apology instead of action | "You're right — fixed in `3a77e02`." |
| "It's much cleaner / significantly faster." | Zero-information intensifier | Replace with the metric, the diff, or delete the sentence. |
| "Will address in a follow-up." | Item exits the ledger | "Deferred: ISSUE-1043 (owner, date, link)." — a follow-up without an ID does not exist. |
| "Per your request, I added a cache." | Nobody owns the decision | "Added the response cache because R4's benchmark showed 40% repeat traffic (`make bench-replay`); reverts cleanly if that assumption breaks." |

### 2.4 Failure diagnostics

| Symptom in the thread | Diagnosis | Fix |
|---|---|---|
| "Can you share the numbers?" | Disagreement was answered at Rung 1 | Re-answer with claim → mechanism → measurement + baseline + command |
| Same comment re-raised in round 2 | Item never reached a terminal disposition | Number the item; assign `ACCEPTED`/`DECLINED`/`DEFERRED`/`INSUFFICIENT` explicitly |
| Reviewer restates; author restates; no movement | Both sides trading intent, no artifact | Ask for the artifact that would settle it; if none exists, mark `INSUFFICIENT` with owner + date |
| Thread becomes a terminology argument | A premise was never grounded | Ground the premise or label it `JUDGMENT` with its criterion |
| PR approved but concerns visibly unresolved | Approval was social, not evidential | Fold the LoR into the PR body; approvals should cite the artifact they verified |
| Concern resurfaces after merge | `DEFERRED` with no issue ID, owner, or date | File it before merge; deferred items must be linkable |
| Author answers in DM, edits nothing | Resolution not patched back into the artifact | Write the disposition into the PR body or the artifact's thread |
| Reviewer feels ignored; tone degrades | Move 1 skipped — no acknowledgement | Restate the concern in the author's words before evaluating it |

**Related dispatchers.** Convert the disputed claim into a falsifiable baseline with the [3-Part Proposal Engine](../../kirby-fitzpatrick-3part-proposal-engine/SKILL.md); frame the disputed surface for the reviewer's actual stance with the [3-Persona Stakeholder Framer](../../kirby-fitzpatrick-3persona-stakeholder-framer/SKILL.md); audit your own comment as a zero-context artifact with the [Cold Reader PR Auditor](../../kirby-fitzpatrick-cold-reader-pr-auditor/SKILL.md); put the familiar term in the reader's doorstep position with [Topic-Comment Elaboration](../../kirby-fitzpatrick-topic-comment-elaboration/SKILL.md); and lead each response with the disposition rather than the narrative, using [Here's Why Inversion](../../kirby-fitzpatrick-heres-why-inversion/SKILL.md).

---

## 3. Engineering Application Scenarios

### 3.1 Code Reviews — Answering a Contested PR Thread

**Author-side procedure (one round, in order):**

1. **Snapshot and number.** Copy the reviewer's comments into a local list as `R1…Rn`. Do not answer from memory; the thread is the input to the response, not a backdrop to it.
2. **Classify each item:** *blocking* (correctness, safety, scope, contract) vs. *non-blocking* (style, future work, curiosity). Blocking items must terminate with `ACCEPTED` or with `DECLINED` + benchmark. Non-blocking items may legitimately end in `DEFERRED`.
3. **Decide the disposition before writing.** Writing first produces rhetoric; deciding first produces a ledger. If you find yourself arguing at Rung 1, you do not yet have a disposition — you have `INSUFFICIENT`, so go produce the artifact.
4. **Write the three moves per item:** acknowledge → concrete change (with `path:line` and test) → benchmark data for disagreements.
5. **Post one structured response, not five replies.** A single numbered letter is what a future reader can audit; interleaved replies are archaeology.
6. **Push the accepted diffs first, then cite the SHA.** A disposition referring to a commit that does not exist yet is a promise.
7. **Patch the ledger into the PR body, then collapse the threads.** The body is the durability layer.

**Before — Rung 1, no artifact, no terminal state:**

```text
A: "Fixed."
A: "I don't think batching will be slow — we do this elsewhere."
A: "That's just a style preference, I'd rather keep it."
```

**After — one letter, four dispositions, all terminal:**

```markdown
## Response to review (PR #812 @ 3a77e02)

| # | Reviewer item | Disposition | Evidence |
|---|---|---|---|
| R1 | "Per-row scoping should go through the repo layer" | ACCEPTED | `handler.ts:88` → `TenantRepo.findOrders()`; `TestReconcileTenantScope` |
| R2 | "Batching will be slow on large tenants" | DECLINED | p95 480→210 ms @100k; no regression @1M; `make bench-reconcile`; baseline `main @ 4f9c1ab`, median of 5, ±6 ms |
| R3 | "Error message is too terse for operators" | DEFERRED | `ISSUE-1043`, owner: reviewer, due 2026-05-02 |
| R4 | "Where is the query-count guard?" | ACCEPTED | assertion added at `reconcile_test.go:214` (same commit as R1) |

**R2 detail** — concern acknowledged: batching moves work into a wide `IN` clause.
Mechanism: one batched query replaces 1,240 per-invoice fetches (`handler.go:88`).
Scaling check: `prod-sample-1m` p95 214 ms (baseline 501 ms). Reverses if tenant
invoice counts exceed 1M — untested today, would reopen as a new item.
```

**Acceptance rule.** A blocking item may end only in `ACCEPTED` with a diff, or `DECLINED` with a benchmark including a named baseline and a reproducible command. A suspicion from the *author's* side must be phrased as a request for the reviewer's measurement shape (parity), never as a verdict.

### 3.2 PR Descriptions — Folding the Letter of Response into the Change Record

The PR body is where a review's outcome outlives its thread. The diff records mechanics; the body records **intent, evidence, and dispositions**. Six months later the diff is still readable and the reasoning is not — unless the LoR was patched in.

```markdown
## Objections raised and how they were resolved
| # | Objection (reviewer) | Disposition | Evidence in this PR | Reverses if |
|---|---|---|---|---|
| R1 | per-row scoping bypasses the repo layer | ACCEPTED | `3a77e02` · `handler.ts:88` · `TestReconcileTenantScope` | — |
| R2 | batching will be slow on large tenants | DECLINED | bench p95 480→210 ms @100k; 214 ms @1m; `make bench-reconcile RUNS=5`; baseline `main @ 4f9c1ab` | tenant invoices > 1M |
| R3 | operator-facing error text | DEFERRED | `ISSUE-1043` (owner, due 2026-05-02) | — |
| R4 | missing query-count guard | ACCEPTED | assertion `reconcile_test.go:214` | — |

## Evidence (claim ↔ mechanism ↔ measurement)
| Claim | Mechanism | Artifact |
|---|---|---|
| p95 480 ms → 210 ms | N+1 fetch → single batch (`handler.go:88`) | `make bench-reconcile WORKLOAD=prod-sample-100k RUNS=5` |
| queries 1,240 → 41 | same | `reconcile_test.go:214` |
| payload unchanged | same assembly path | `scripts/diff_reconcile.py` → 0 row diffs @100k |
```

**Binding rules that keep the block auditable:**

- **Round-trip consistency.** The dispositions must agree with the evidence table and with the code at the cited SHA. "No regression" above a `p95 +8%` row destroys trust in every other number, including the ones that were right.
- **Every cited command must exist in the repository.** If `make bench-reconcile` is missing, either add it or delete the number — an un-runnable citation is worse than no citation, because it looks verified.
- **Name the baseline as a ref.** "Before" without a SHA, branch, or tag is a memory, not a baseline.
- **Label single runs as single runs.** Otherwise they read as measurements.
- **Deferred ≠ dropped.** A `DEFERRED` row without ID + owner + date is rejected at merge time.
- **The PR body is not the reviewer's mailbox.** Write for the next reader, not to win the thread: dispositions, not rebuttals. Anything that is only an argument belongs in the closed thread.
- **Order the block by item number, not by importance.** The reader is cross-checking against the thread; reordering makes the audit impossible.

### 3.3 Architecture RFCs / ADRs — The Objections & Dispositions Ledger

An ADR outlives its reviewers, its authors, and often the team. Objections raised during review are the most valuable material in it — they are the *rejected alternatives* seen from the other side — and they are the first thing an ADR throws away when it is written as a summary instead of a ledger.

**The Objections & Dispositions ledger.** Every review objection gets a row: class, disposition, evidence, and the amendment it caused. Dissent is recorded, not suppressed — a recorded dissent with a criterion is a future signal; a suppressed one is an incident nobody predicted.

| # | Objection as raised | Class | Disposition | Evidence / amendment | Owner · date |
|---|---|---|---|---|---|
| O1 | "Batch size will exceed Postgres parameter limits" | correctness | `INSUFFICIENT` | No test above 10k IDs; batch chunking added at 5k (`handler.go:104`) pending a load run | author · blocking |
| O2 | "Two write paths now bypass `TenantRepo`" | safety | `ACCEPTED` | ADR §Decision amended; `grep -rn "from orders" src/` → 0 hits outside repo layer | author |
| O3 | "Migration needs a 3-week freeze" | cost | `DECLINED` | Dual-read spike: 4 days measured (`spikes/dual-read/README.md`); freeze not required | EM |
| O4 | "Read replicas would be simpler" | design | `DECLINED` | Rejected alternative §Cost: replica lag p99 1.8 s vs. budget 250 ms (metric `replica_lag_p99`) | author |

**Rules for the ledger:**

- **A `DECLINED` objection must name the axis and the measurement that decided it** — cost, latency, blast radius, contract stability. "Team agreed" is not evidence; it is the decision without its grounds.
- **An `INSUFFICIENT` objection is a blocking marker.** State who resolves it, by when, and exactly which artifact closes it. Silent unknowns become incidents.
- **Amendment, not deletion.** When an objection changes the decision, show the amendment: what the §Decision or §Consequences text now says and why. An ADR that reads as if nobody ever objected teaches the next reader nothing.
- **Record dissents with the condition that would prove them right** (*"reverses if tenant invoices exceed 1M"*). This is the ADR's expiry mechanism — it tells a reader in twelve months when the decision has lapsed instead of leaving them to find out in production.
- **Pin the environment of every measurement.** *"In production"* is not an environment: region, cluster, runtime version, and measurement source.
- **Objections answered verbally are reconstructed here** — decision, date, deciders, evidence, reversal condition. An RFC that cannot be understood without its originating meeting is private notes with a title.

```markdown
# ADR-014 — Batch reconciliation queries

## Review objections and dispositions
| # | Objection | Disposition | Evidence | Status |
|---|---|---|---|---|
| O1 | Postgres parameter limits at high batch size | INSUFFICIENT | chunking at 5k IDs (`handler.go:104`); load run at 50k pending | blocking — author, before merge |
| O2 | bypassed repo layer | ACCEPTED | ADR §Decision amended; `grep -rn "from orders" src/` → 0 | closed @ 3a77e02 |
| O3 | requires a 3-week freeze | DECLINED | dual-read spike measured at 4 days | closed |
| O4 | read replicas would be simpler | DECLINED | replica lag p99 1.8 s vs. 250 ms budget | closed |

## Decision
Issue one batched query per batch (chunked at 5k IDs), retaining the payload assembly path.

## Consequences
- Measured: p95 480 ms → 210 ms @100k; 214 ms @1m; queries 1,240 → 41
  (`make bench-reconcile`, baseline `main @ 4f9c1ab`, same runner, median of 5).
- Cost of inaction: 4 timeouts in 3 weeks (INC-4501, INC-4522); query volume grows linearly
  with invoice count, so p95 worsens with traffic, not with load.
- Reverses if: A4 (consumer audit) or O1 (batch-size limits) invalidates batching.
  Rollback: revert this commit; no schema or config migration.
```

---

## 4. Verification Checklist

- [ ] **Every review item reached a terminal disposition.** Each numbered item is `ACCEPTED`, `DECLINED`, `DEFERRED`, or `INSUFFICIENT` — no item was left as an acknowledgement, a promise, or silence, and the open-item count strictly decreased each round.
- [ ] **Acknowledgement precedes evaluation in every response.** Each item restates the reviewer's concern in the author's words (comprehension evidence) before it accepts, declines, or defers; nothing was answered with a verdict against a strawman version of the comment.
- [ ] **Every acceptance cites a concrete change at a named commit.** Dispositions point at `path:line` at the PR head SHA, with the test or assertion that now covers the behavior; every cited command actually exists in the repository and reproduces the stated result.
- [ ] **Every disagreement carries benchmark data with a named baseline.** Claim → mechanism → measurement, plus baseline ref, workload, runner, and runs/dispersion. Unmeasured disagreements were downgraded to `INSUFFICIENT` with a named measurement, owner, and date — no one won an argument by assertion, authority, or "should be fine".
- [ ] **The letter of response is patched into the durable artifact.** The disposition ledger lives in the PR body or the ADR (§Objections and dispositions), not only in the thread; every `DEFERRED` row names an issue ID, an owner, and a date, and every `DECLINED` row names the condition that would reverse it.
- [ ] **Tone and ownership audit passes.** No apology-as-substitute-for-action, no unearned authority markers (*obviously*, *clearly*, *much cleaner*), no zero-information intensifiers, and no laundering of decisions onto the reviewer (*"per your request"*, *"happy to change if you feel strongly"*) — each such instance was replaced by a fact, a diff, or a number.