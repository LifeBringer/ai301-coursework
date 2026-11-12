# Contribution: Home summary refresh exceeds its 60s deadline: snapshot re-runs full status_open_decisions fold per task

Path: `beat-2-contribution/contribution-story.md`

**Contribution Number:** [1]
**Student:** Roy Alda
**GitHub Username:** LifeBringer
**Issue:** https://github.com/kunchenguid/firstmate/issues/6257
**Pull Request:** https://github.com/kunchenguid/firstmate/pull/7376

I built PR #7376 for this course, around the time Beat 2 opened, and before I installed the Beat 2 tools (scout, voice guard, ship-gate, PR monitor). Staff approved this path. I ran every Beat 2 tool on 2026-11-11 and 2026-11-12, after the PR existed, under my extension. I first ran them with the PR and the issue untouched. Later on 2026-11-12 I acted on what they found: I posted my claim and reproduction on #6257, rebased #7376 onto current main through no-mistakes, and fixed its description.

---

## Toolkit

Every story section below, from Why I Chose This Issue through Pull
Request, ends with a Toolkit block: which tool did that phase's work
and how you used it, what it produced, what difficulties or
shortcomings it presented, and what you changed or would change
because of them.

A Toolkit block looks like this (copy it into each story section and
fill it there):

```text
**Toolkit**

- Tool: [which tool did this phase's work, and how you used it]
- What it produced: [described in a sentence or a short excerpt]
- Difficulties: [what fought you or fell short, or "none met"]
- What changed: [the patch, revision, or workaround, what you would
  update, or "nothing"]
```

**Unit 5: built the scout, ran two scans, and graded #6257 directly (2026-11-12)**

I wrote my fit profile (Python, TypeScript, and the shell tooling I run every day, firstmate included) and a strategy with three `gh search issues` queries, ranked firstmate first and newest first. The first scan graded five firstmate issues labelled that same day and never saw #6257: the firstmate query capped at 100 of 485 open `ready-for-pr` issues. I narrowed that query to the symptom my own home logs (`deadline`) and added drop rules for consolidation-shaped titles and the `existing-pr` label. The second scan found #6257 but ranked it 11th, and I left the ordering alone on purpose, because tuning it until a known answer lands on top fits the strategy to one issue, not to my profile.

**Unit 6: voice guard, upstream rules, rubric patch, and repo memory (2026-11-11 to 2026-11-12)**

I installed the voice guard inside my course folder instead of user-wide, because a global hook would also read what my firstmate agents post in my other repos. The wiring test proved it: `gh issue comment --help` was stopped from the course folder and ran normally from `/tmp`. The guard's first real check stopped my claim draft on my own new rule (a claim that points to an existing PR says what it changes in one or two sentences), and I cut the draft to pass. I applied the scope patch to my rubric, removed a leftover Path Review rule from my Available work check, and wrote `CLAUDE.local.md` in a separate course clone so it never loads in my live firstmate home. The guard then held every outgoing word: it passed my posted claim comment (logged as a hook pass), stopped my no-mistakes intent twice (no AI disclosure, then evidence linked instead of shown), and passed the PR body edit.

**Unit 7: ship-gate and manifest, run on the existing branch (2026-11-12)**

My manifest started with three checks from CONTRIBUTING.md: `bin/fm-lint.sh`, and `bin/fm-test-run.sh` on `fm-home-summary-refresh` and `fm-classify-decision-key`. I did not register the gate hook, because it only gates `gh pr create` and my PR already existed, so I ran `check_gate.py --branch` by hand against PR head `fd404cbd`, twice. Run 1 said STOP: it diffed against my clone's stale local `main` (516 files), so pr-precheck never saw the fix, and it caught that my manifest skipped `bin/fm-doc-audience-check.sh`, which CONTRIBUTING.md requires after doc changes. I passed `--base upstream/main` and added that check. Run 2 passed all four checks and still said STOP on one row, Plan fidelity, because my PR description claims the docs line says more than it does. That one needed a PR edit. Later that day I rebased the branch onto main a9ae3c0c and re-raised it through no-mistakes with a clean intent, which regenerated the description, then made one hand edit so `Fixes #6257` links the issue. Run 3 on the new head daaa6660: all four checks ok and **PASS**. The only miss was the preferred Calibrated claims check: my intent cites my 480-fold fuzz, but the PR's attached evidence is the pipeline's own fuzz, and two bullets disagree on the run count.

**Unit 8: PR monitor and playbook (2026-11-12)**

I filled the playbook from real numbers: five recently merged outside PRs got a first maintainer response in 0.3 to 3.4 hours, while #7376 waited 21 days. The monitor's state read caught something I had not seen: the branch now conflicts with main in `bin/fm-classify-lib.sh`. The full run drafted a "do not post" wait note, since the 7-day stall threshold after the `full-ci` label lands on 2026-11-13. I checked the two things it said it could not see (the failing check is the pre-label `Full suite` run, and the post-label CI run is still waiting for fork approval), agreed with it, and sent no nudge. The conflict it found was the real blocker, so instead of nudging I fixed it: rebased, re-raised through no-mistakes, and left CI waiting for the maintainer's approval again. A second state read after the push showed the PR mergeable again, with the thread quiet for 5 days apart from my own activity.

---

## Why I Chose This Issue

I didn't find this issue with a search. I hit it. My own firstmate home logged 118 "refresh exceeded its 60-second deadline" failures between 2026-09-30 and 2026-10-14, 82 of them on 2026-10-14 alone. I reproduced it that day, then found #6257 already filed by deorozindo with the same symptom and a per-function profile, and triaged `ready-for-pr` by the maintainer with "No open helping PR". It fit what I wanted from a first contribution: a bounded behavior fix in a tool I run every day, on my own machine, with a timer as the verification (exit 124 at 60s, or not). I wanted to get better at performance work in plain Bash.

Here's the part I got wrong. By the time I opened #7376 on 2026-10-16, two other PRs were already open on this issue: #6464 by luigia (2026-10-03) and #6980 by zico-lle (2026-10-10). I never checked the issue timeline, and my PR description never names #6257, so nothing connected them. My scout caught it in one call on 2026-11-12. Its verdict on #6257 was **reject**, on Available work, and the re-grade after the unit 6 rubric patch said the same: "Two open pull requests already implement this fix, so the work is taken." I'm keeping #6257 as my contribution anyway, because the PR is built, staff approved this path, and the honest record is more useful to me than a fresh pick. My claim, with a reproduction anyone can rerun, is here: https://github.com/kunchenguid/firstmate/issues/6257#issuecomment-6830359873 (posted 2026-11-12). It names #6464 and #6980, says I missed them, and asks the maintainer which direction to keep, with an offer to close mine.

My read on the scout after a real wild search: the grading is the strong half and the search is the weak half. Grading found two competing PRs that an empty Development sidebar hid, because it followed the timeline cross-references. The search fell short twice. Newest-first ordering on a repo with hundreds of fresh `ready-for-pr` issues only grades what a triage bot labelled that morning, and a body keyword like `deadline` matches a CI job's "Deadline: Nov 19". If I ran it again I'd scope queries by symptom from the start, drop `existing-pr` in ranking (it's a free field), and grade any issue I'm about to work on before writing code, not after.

**Toolkit**

- Tool: issue-scout. I ran two full scans (`claude "issue-scout: run a scan"`) and graded #6257 directly twice, before and after the unit 6 rubric patch, all on 2026-11-12, after #7376 already existed.
- What it produced: scan 1 accepted #8388 and rejected four, without seeing #6257. Both direct grades rejected #6257 on Available work: "timeline cross-references open PRs #6464 (luigia, 2026-10-03) and #6980 (zico-lle, 2026-10-10), both within 90 days". Scan 2 accepted #4209 (difficulty: ambitious) and ranked #6257 11th.
- Difficulties: the firstmate query hit its 100-result cap, newest-first ranking buried older triaged issues, my rubric still carried a Path Review rule telling it to ignore other people's claims, and three of five graded issues were redesigns that my old Bounded check only rejected by stretching its wording.
- What changed: I narrowed the firstmate query to the `deadline` symptom, added drop rules for consolidation titles and the `existing-pr` label, swapped Bounded for the scope-feasible check, and told Available work to search the timeline. All dated in `scan-notes.md`.

---

## Understanding the Issue

**Problem Description**

On a home with many tasks and long status histories, the home summary refresh can't finish inside its 60-second deadline. Every refresh gets killed, so the published summary goes stale. The issue measured 88s on a home with 12 secondmates and 2,513 status lines.

**Expected Behavior**

`bin/fm-home-summary-refresh.sh` publishes a fresh `state/home-summary.json` well inside the default `FM_HOME_SUMMARY_TIMEOUT` of 60s, and keeps doing so as status logs grow, without raising the deadline.

**Current Behavior**

The refresh exits 124 with `fm-home-summary-refresh: refresh exceeded its 60-second deadline` while its child `bin/fm-fleet-snapshot.sh --secondmate-home-summary` is still collecting. The last complete summary stays intact, so nothing breaks loudly, but it stops updating.

**Affected Components**

- `bin/fm-home-summary-refresh.sh`, which owns the 60s bound, validation, and atomic publication.
- `bin/fm-fleet-snapshot.sh --secondmate-home-summary`, the producer. It reads task state in batched workers, then composed each task's row one task at a time.
- `bin/fm-classify-lib.sh`, where `status_open_decisions` folds each task's whole status history through `status_line_note`, `_fm_decision_key`, `_fm_key_at_note_head`, `_fm_decision_drop` and `_fm_decision_fold_line`.

**Toolkit**

- Tool: my firstmate agent fleet's scout, which read the producer and an instrumented copy of it in a lab home on 2026-10-14, plus the issue's own xtrace profile. On 2026-11-11 I distilled the subsystem (the 3 files, how they connect, and their 2 test suites) into `CLAUDE.local.md` in my course clone, from my own reading rather than a separate orientation run.
- What it produced: a phase timing for the failing run. The bounded reads finished at 23.4s, serial row composition started at 23.5s, and only 6 of 12 rows were done when the deadline hit. The largest task's fold alone ran from 36.2s to 57.7s.
- Difficulties: the issue blamed the snapshot for not using the incremental fold, and that pulled me toward a cursor first. The timing showed the bigger problem was the serial composition phase, which the issue didn't name.
- What changed: I scoped the fix to the serial phase and the fold's forks, and left the cursor idea out (see Solution Approach).

---

## Reproduction Process

**Environment Setup**

- macOS 27.0.1 on an 8-core Apple M2 (4 performance, 4 efficiency cores).
- Stock `/bin/bash` 3.2.57, which is what firstmate's scripts run under on macOS. The issue was filed from a Linux container with bash 5 at firstmate 9296f9b9, and I didn't test that setup.
- firstmate base 230db82 for the first PR head, upstream main bf05a4fc after the 2026-11-06 rebase, and main a9ae3c0c for the public reproduction and the 2026-11-12 rebase.
- ShellCheck 0.11.0 and actionlint 1.7.12 (the versions `bin/fm-lint.sh` pins), and no-mistakes v1.84.0 for the PR pipeline.
- Disposable lab homes made with `bin/fm-lab-home.sh`, so nothing touched my real fleet. tmux and no-mistakes were stubs on PATH in the lab, so endpoint probes did not hit real panes. Earlier lab runs used Herdr 0.9.3 sessions from `bin/fm-herdr-lab.sh`.
- Load: some runs used a quiet machine, and some ran next to other lab scenarios to emulate a busy host (background-QoS busy loops via `taskpolicy -b`). For the public reproduction I emulated contention the same way: the test at `taskpolicy -b` next to 4 `taskpolicy -b` busy loops. Prerequisites: `python3` and `jq`.

Challenges: the lab home starts empty, so I had to seed incident-shaped status histories. The fixture is now public as `seed_fleet_home` plus the incident seeding in `tests/fm-home-summary-refresh.test.sh`. The other challenge was my locked Mac. Every `gh` call failed with "Secrets are unavailable from Keychain" (-25308) whenever the screen locked, which broke CI observation on 2026-10-16 and again on 2026-11-12 while I wrote this. The fix was unlocking and rerunning. I didn't change the credential setup.

**Steps to Reproduce**

These are the public steps from my claim comment, rerun on 2026-11-12 against current main.

1. In a checkout of main, fetch #7376 (`git fetch https://github.com/kunchenguid/firstmate.git pull/7376/head:pr-7376`) and build `tests/without-rendezvous.test.sh` from its test file with the `sed` in my comment. It drops the four-row rendezvous case (it fails on main by design and stops the script) and times only the incident-shaped refresh: 12 in-flight tasks, about 508 status lines each, default 60s deadline.
2. Quiet: `bash tests/without-rendezvous.test.sh`
3. Contended: start 4 `taskpolicy -b sh -c 'while :; do :; done' &` busy loops, run `taskpolicy -b bash tests/without-rendezvous.test.sh`, then stop the loops.
4. Observed on main a9ae3c0c under contention: `not ok - the incident-shaped fleet did not publish within the default deadline`, stderr `refresh exceeded its 60-second deadline`. Quiet, main published in 37s.

**Reproduction Evidence**

- **Commit showing reproduction:** https://github.com/LifeBringer/firstmate/blob/1cec2369620fc744e41e457faca761f9beb04161/.no-mistakes/evidence/fm/firstmate-home-summary-timeout/incident-12x505-quiet-base-vs-head.txt (the original lab evidence for the first head). The current, rerunnable reproduction is my posted comment: https://github.com/kunchenguid/firstmate/issues/6257#issuecomment-6830359873
- **Screenshots/logs:** the same commit holds the loaded run (`incident-12x505-base-vs-head.txt`: base exit 124 at 61.2s) and the deadline guard (`deadline-guard.txt`). My own home's direct run on 2026-10-14 also exited 124, with `fm-fleet-snapshot.sh --secondmate-home-summary` as the terminated child. That log is private.
- **My findings:** the failure needs contention plus long histories. On current main a9ae3c0c the incident case published in 37s quiet (1 run) and missed the 60s deadline under contention (1 run). Head fd404cbd published in 6s quiet and 20s contended. Its first contended run stopped earlier, at the existing case `the bounded writer did not expose its worker process group`, and the passing run later failed another existing case, `a dead publication lock wedged publication`. Both cases predate #7376 and look load-sensitive. My first lab runs on base 230db82 told the same story: 34.8s uncontended once, exit 124 every time under contention. The 1s per-child bound control from those runs (the bound was not the cause) is in my private notes, not the posted evidence.

**Toolkit**

- Tool: the repro-check skill on 2026-11-12, four rounds: two on my first claim and repro drafts, then two on the single combined comment I posted.
- What it produced: rounds 1 to 3 rejected, round 4 accepted all six checks. Round 1 failed Actionable environment, Rerunnable steps, and Bounded honest outcome. Round 2 passed environment and failed steps ("`seed_fleet_home` depends on test-harness helpers") and outcome ("a causal claim resting only on private runs"). Round 3 caught that my table merged two head runs into one, and that my timings came from a timer my steps never showed.
- Difficulties: the tool was right on every point. My first report never said my environment differed from the issue's (Linux, bash 5), it described a fixture a stranger couldn't rebuild, and it stated a cause from runs I never published. It also flagged that my evidence names pre-rebase head 77b0325, which no longer resolves on GitHub.
- What changed: I replaced the private fixture with a public command built from the PR's own test, added a timer that measures only the incident refresh, gave a run count for every result, named `python3` and `jq`, and stated how the fixture differs from the issue's home. Then I posted it through the voice-guarded session.

---

## Solution Approach

**Analysis**

`bin/fm-home-summary-refresh.sh` bounds the whole refresh at 60s. Inside it, `bin/fm-fleet-snapshot.sh` reads each task's current state in workers batched by `FM_SNAPSHOT_LOCAL_READ_CONCURRENCY` (default 8), and those reads stayed inside their 10s per-child bound. After the reads, though, `task_json_lines` composed every task row one task at a time. Each row re-folded the task's whole append-only status history through `status_open_decisions` in `bin/fm-classify-lib.sh`. That fold called `status_line_note`, `_fm_decision_key`, `_fm_key_at_note_head`, `_fm_decision_drop` and `_fm_decision_fold_line` through command substitutions, so it forked four to five subshells per decision transition (about 1,300 for my largest log). Cost grew with task count times history length, ran serially, and had no bound of its own, so a busy host pushed it past 60s.

**Proposed Solution**

Two changes. First, run each task's whole-history fold and its row composition inside that task's own observation worker, so they overlap the bounded read and run in parallel across tasks. Second, let the fold helpers fill an optional output variable with `printf -v`, so the whole-file fold forks nothing per line. Called without the variable, every helper prints the same output as before.

What I'm deliberately not changing: the 60s deadline, the per-child read bounds, validation and atomic last-good publication, the refresh lock, the default concurrency of 8, and the published content (byte-identical between base and head on my test homes). Not solved here: collection cost still grows with unbounded history (about 2ms per line in bash 3.2), `fm-crew-state.sh` still folds the same history again, and the reporter's pending-reply retention follow-up is out of scope. Those are follow-ups.

The reporter suggested cursor-based scanning, and the owner's triage named a "correctness-preserving incremental/cache/retention" shape while noting the snapshot keeps the whole-stream fold for keyed-decision correctness. I kept the whole-stream fold. `status_open_decisions_incremental` persists a per-file byte cursor and open set as a side effect, and the snapshot is a read path with its own test that it doesn't mutate operational state. The worker move plus the fork-free fold fixed the deadline with no new state, and byte-identical output is easy to prove.

**Implementation Plan**

I wrote this plan on 2026-11-11 from the finished diff, because the PR already existed. It records the change rather than predicting it.

1. `bin/fm-fleet-snapshot.sh` (+210 -192): run each local task's `status_open_decisions` fold and `task_json_row` composition inside its observation worker, still batched by `FM_SNAPSHOT_LOCAL_READ_CONCURRENCY`. `task_json_lines` only merges and sorts the finished `.row` files. A failed row fails the snapshot instead of dropping that task.
2. `bin/fm-classify-lib.sh` (+87 -57): add the optional out-variable form to `status_line_note`, `_fm_decision_key`, `_fm_key_at_note_head`, `_fm_decision_drop` and `_fm_decision_fold_line`, and use it in `status_open_decisions`.
3. `docs/configuration.md` (+1 -1): the `FM_SNAPSHOT_LOCAL_READ_CONCURRENCY` line now says the status-history fold and row composition also run under that cap. The PR description mentions this file only inside the first What Changed bullet, and it overstates it: it says the entry describes batched workers and history growth, which it does not. The ship-gate caught that on 2026-11-12.
4. `tests/fm-home-summary-refresh.test.sh` (+162): three new cases. A collection that outlives its deadline exits 124 and keeps the last summary. Four task rows compose at the same time. A 12-task fleet with hundreds of decision transitions per task publishes inside the default deadline.

Deviations, recorded as they happened:
- 2026-10-15: the no-mistakes review added two commits after the fix. One rewords the help text and comments to describe batched workers and the remaining history cost. The other renames the per-task row temp file from `$id.row.json` to `$id.row`, because task ids `a` and `a.row` could collide.
- 2026-11-06: on the rebase, upstream had added a 6th `[timed]` argument to `_fm_decision_fold_line`, so my out-variable moved to the 7th position.
- 2026-11-12: the branch conflicted with main again in `_fm_decision_fold_line`, because upstream extended the timed view with a deferral column and kept an earlier epoch for republished keys. I rebased onto main a9ae3c0c, kept my `_fm_decision_fold_emit` helper and 7th out-variable argument, and merged both sides' `local` lines. Fuzzed folds then matched main in 480 of 480 comparisons. no-mistakes re-raised the branch as head daaa6660.
- The 2026-10-16 pipeline listed open findings: collection cost isn't bounded for unbounded history (warning), and help text and comment wording on batching (infos). The wording was fixed in the second review commit. The cost warning is still a follow-up, and the new description's Risk Assessment and intent say so.

**Toolkit**

- Tool: the plan-check skill on 2026-11-12, grading `plan.md` (the text above) and a local plan comment for #6257. I ran it three times and never posted the plan comment.
- What it produced: round 1 rejected on one row, Thread and conventions, because my comment never answered the owner's "incremental/cache/retention" direction. The other four required checks passed. Round 2 refused to grade: "Do not grade plans for an issue you have not claimed." Round 3, after my claim and repro were posted, was **accept**: all five required checks passed. The preferred Calibrated claims check failed, because the plan cites results (the 1s bound control, byte-identical homes) that aren't in the posted evidence, and my comment quoted the 20s head result without its earlier stopped run.
- Difficulties: the refusal is correct under the wild scope. I have no claim on #6257, my scout rejected the issue, and the plan was written after the build, which the tool said makes it "a record of the PR, not a plan to build from". The accept came with a fair note: house rule 1 says someone else's posted plan comes first, and no rubric check reads it.
- What changed: I added the triage-direction paragraph above and the pending-reply out-of-scope line, posted the claim and repro on #6257, and then added the earlier-PRs and evidence-limits sections to `plan.md`. The plan comment stays unposted, because my posted comment already says the same thing.

---

## Testing Strategy

Two observable checks.

1. **The reproduction re-run against the built change, before and after.** Same incident-shaped lab home, same command, default 60s deadline:

   | Run | Base 230db82 | Head 77b0325 (pre-rebase) |
   |---|---|---|
   | Quiet machine | exit 124 at 61.4s | published in 11.4s |
   | Under concurrent load | exit 124 at 61.2s | published in 51.5s |

   Deadline guard at head with `FM_HOME_SUMMARY_TIMEOUT=3`: exit 124 after 3s, last summary byte-identical, no temp files, no orphaned workers. Base and head published byte-identical summaries on a history-heavy home (4 x 505 lines) and an edge-case home. After the 2026-11-06 rebase, head fd404cbd published in 11.4s, 9.0s and 4.0s at normal priority, while upstream main published once in 48.5s and once exited 124.

   Re-run on 2026-11-12 with the public steps from my claim comment (main a9ae3c0c vs pre-rebase head fd404cbd, incident refresh only):

   | Run | main a9ae3c0c | head fd404cbd |
   |---|---|---|
   | Quiet machine (1 run each) | published in 37s | published in 6s |
   | Contended (background QoS + 4 busy loops) | missed the 60s deadline (1 run) | published in 20s (run 2; run 1 stopped at an existing case first) |

   The no-mistakes test step on the rebased head daaa6660 measured the same direction: the real refresh published in 7s on head vs 39s on base, and collection took 4s vs 26s.

2. **The repo's own checks.** On head fd404cbd before the rebase:
   - `bin/fm-lint.sh`: exit 0 (ShellCheck 0.11.0, actionlint 1.7.12, 3 workflow files valid).
   - `bin/fm-test-run.sh tests/fm-home-summary-refresh.test.sh`: 24 ok, exit 0 (69.3s by hand, 116s and 108s inside the two ship-gate runs). This includes the three new cases.
   - `bin/fm-test-run.sh tests/fm-classify-decision-key.test.sh`: 23 ok, exit 0.
   - `bin/fm-doc-audience-check.sh`: exit 0 (141 surfaces, 1193 local links), because the PR edits `docs/configuration.md`.
   - On GitHub: `Test coverage guard`, `Repo invariants` and the no-mistakes body check pass on fd404cbd. The label-gated `Full suite` hasn't run yet (see Pull Request).
   - After the rebase, on head daaa6660: `bin/fm-lint.sh` exit 0, `bin/fm-doc-audience-check.sh` exit 0, and 8 of 8 suites passed in one `bin/fm-test-run.sh` call (`fm-home-summary-refresh`, `fm-classify-decision-key`, `fm-classify-corr-token`, `fm-fleet-snapshot-view`, `fm-fleet-snapshot-contribution-input`, `fm-wake-drain-open-decisions`, `fm-wake-drain-open-decisions-cursor`, `fm-watch-open-decisions`), because `fm-classify-lib.sh` changed under every caller of the fold.
   - Fold equivalence after the rebase: 480 of 480 whole-file, per-line, timed, and out-variable folds over 40 fuzzed status logs (2,703 lines, with stamps, `until=` tags, and both key positions) matched main byte for byte.
   - On GitHub for daaa6660: Greptile passes. CI and the no-mistakes body check are at `action_required`, waiting for the maintainer to approve the fork workflows.

One honest wrinkle: during the original pipeline run, `fm-home-summary-refresh.test.sh` failed one assertion once ("the status-triggered ledger differed from the real fresh producer") while I was saturating the CPU with other lab runs. Run alone it passed all 24. It hasn't failed in any run since, including today's two. The rebased pipeline run on 2026-11-12 hit a different existing case, `a dead publication lock wedged publication`, in 2 of 4 runs while the host load average was about 9. It passed at load 5 to 6. The case waits only 10s for a background publication. It isn't part of this change, so I kept it out of the PR and noted it as a follow-up.

**Toolkit**

- Tool: no-mistakes v1.84.0 (the pipeline firstmate requires for outside PRs), which ran review, test, document and lint before the push, and `bin/fm-test-run.sh` by hand on 2026-11-12.
- What it produced: four live validation scenarios marked pass in the PR body, with evidence files committed to my fork, and today's 24 ok and 23 ok runs. On 2026-11-12 the rebased run's test step reported head 7s vs base 39s for the real refresh, plus one warning about that flaky existing case.
- Difficulties: the pipeline's test agent hit its 30-minute budget ("agent run tests timed out after 30m0s") and its lint step reported a warning, both visible in the PR's Pipeline section. The broad local `--changed` run had to be stopped after 19 of 71 scripts to stop loading the host. On 2026-11-12, run 2 of the pipeline stopped at its rebase step because it tried to rebase onto my fork's old head. I approved the step, since the branch was already ahead of main.
- What changed: I kept local verification to the targeted suites CONTRIBUTING.md names, and left broad regression to CI's `Full suite`, which is how the repo splits the work.

---

## Implementation Notes

Entries before 2026-11-11 are reconstructed on 2026-11-12 from my private notes, the PR, and `git log`.

**2026-09-30**

deorozindo filed #6257 and the maintainer triaged it `ready-for-pr`. That same day my own home logged its first "refresh exceeded its 60-second deadline" failure. (Reconstructed.)

**2026-10-14**

My home logged 82 deadline failures that day, 118 in total since 2026-09-30. I reproduced the failure directly with `bin/fm-home-summary-refresh.sh` (exit 124), then moved all further work into disposable lab homes. Instrumented runs placed the time in serial row composition, not the per-child reads. I committed the fix (author date 2026-10-14, now `6d07b9bc` after the rebase). (Reconstructed.)

**2026-10-15 to 2026-10-16**

I forked firstmate and pushed through no-mistakes. Its review found the batching wording overstated and a dotted-id temp-file collision (`a` vs `a.row`), and it committed both fixes. The pipeline opened PR #7376 at 2026-10-16 01:33 UTC. One wrong turn: no-mistakes v1.84.0 stored my credential helper's three approval lines as the PR URL. It only affected display. (Reconstructed.)

**2026-10-16**

CI showed 18 of 19 checks green. The one red check, `Behavior portable serial 2`, was cancelled at its 30-minute job limit. It runs none of my tests and had hit the same limit on main in 3 of the last 4 runs. Re-running it needs write access I don't have. A locked Keychain also broke `gh` auth while I watched CI. (Reconstructed.)

**2026-10-17**

PR observation became unavailable, and I parked the work to focus on another project. (Reconstructed.)

**2026-11-06**

I resumed. no-mistakes refused to start a fresh run because its private mirror had diverged from my rebased branch. I rebased onto main bf05a4fc by hand, resolving one conflict in `_fm_decision_fold_line` (upstream's new `[timed]` argument). The new pipeline run pushed head fd404cbd, but its PR step failed on a Keychain timeout, so the PR body kept the earlier run's narrative. The maintainer reviewed fd404cbd, approved the fork CI runs, and added `full-ci`. (Reconstructed.)

**2026-11-11**

I installed the Beat 2 tools inside my course folder, made a separate course clone, and wrote `plan.md` from the finished diff.

**2026-11-12**

I ran the scout, voice guard, repro-check, plan-check, ship-gate and PR monitor. The scout surfaced the two earlier PRs on #6257. The monitor surfaced the new merge conflict with main. I also found that the `full-ci` CI run on fd404cbd has been stuck at `action_required` since 2026-11-06.

**2026-11-12 (afternoon)**

I decided to act on what the tools found instead of only disclosing it. I built a public reproduction from the PR's own test and ran it on main a9ae3c0c and head fd404cbd, quiet and contended. After four repro-check rounds I posted my claim and repro on #6257 through the voice-guarded session. plan-check then accepted the plan. I rebased onto main, resolved the `_fm_decision_fold_line` conflict, ran 480 fuzzed fold comparisons and 8 suites, and re-raised through no-mistakes with a clean intent. Its rebase step tried to rebase onto my fork's old head and I approved past it. Its test step flagged an existing flaky case, which I left as a follow-up. The PR step regenerated the description, and I made one hand edit so `Fixes #6257` is a real link. The PR now lists #6257 as an issue it closes.

**Toolkit**

- Tool: `git log --format='%ad %h %s'` on the PR branch, `gh pr view` / `gh run list` for PR and CI history, and my firstmate task notes for the pre-course timeline.
- What it produced: the dated log above. Author dates (2026-10-14 and 2026-10-15) survive the rebase, and the commit dates all read 2026-11-06.
- Difficulties: most of this log is reconstructed, because I didn't keep a story file while building. Git's author and commit dates disagree after a rebase, and that's easy to misread.
- What changed: I labeled every reconstructed entry. Next time I'd write the dated note the same day.

---

## Pull Request

**PR Link:** https://github.com/kunchenguid/firstmate/pull/7376

**PR Description:** The title is "fix(bin): keep home-summary collection inside its refresh deadline". Since the 2026-11-12 re-raise, the body opens with `Fixes #6257` and the problem, the fix, what stays unchanged (deadline, read bounds, concurrency, summary content), the follow-ups, the earlier PRs #6464 and #6980, and the reproduction with its numbers, then an AI disclosure. What Changed covers the worker move, the out-variable fold helpers, the three test cases, and the docs line. Risk Assessment rates it low. Testing attaches evidence for head vs base timing, published summaries, the fold fuzz, and three targeted test runs, including the two where the existing flaky case failed under load. Pipeline shows each step of the new run.

Before 2026-11-12, the description had four defects. Each is now fixed:
- It never named #6257. Now it opens with `Fixes #6257`, and GitHub lists #6257 as an issue the PR closes. The generator first wrote it in backticks, which breaks the link, so I made one hand edit and left the attestation untouched.
- The Intent section held text pasted from my private agent instructions and consent lines. The new intent is written for the maintainer.
- The Pipeline section was from the 2026-10-16 run, including a Lint block that read "exit code 1". The new run's Pipeline shows lint passed.
- It said the `docs/configuration.md` entry describes batched workers and history growth. The new text says the entry was updated to match the new behavior, which is what the one-line diff does.

**Maintainer Feedback:**

- 2026-11-06: The maintainer's fleet ("Speaking as Kun's firstmate") reviewed fd404cbd against main bf05a4fc, approved the pending fork CI runs, and reported the gates green: coverage guard, repo invariants and no-mistakes passed, the attestation binds the head, live validation 4/4, Greptile 5/5. Contract class: restore.
- 2026-11-06: Same day: "I've added the `full-ci` label so the full suite runs; if it's green this merges on its own. Nothing more is needed from you." My response: none needed, and I sent nothing.
- 2026-11-12: No new maintainer comments. The `full-ci` CI run on fd404cbd never ran (`action_required`), and the branch had started conflicting with main. My response: I posted my claim and reproduction on #6257 (naming #6464 and #6980), then rebased and re-raised #7376 as head daaa6660 with a corrected description. CI on the new head waits for the maintainer's fork-workflow approval again.

**Status:** Awaiting review. Open, not draft, `full-ci` labeled, mergeable with main as of 2026-11-12, CI on head daaa6660 waiting for maintainer approval.

**Toolkit**

- Tool: the ship-gate (`check_gate.py --branch` with my manifest, run by hand on 2026-11-12 after the PR was opened), plus no-mistakes, which actually gated the PR on 2026-10-16 and 2026-11-06, and the PR monitor (`monitor.py --state`, then a full run).
- What it produced: ship-gate run 1 STOP on four rows (two were my setup, stale base and a missing manifest check). Run 2: four manifest checks ok (lint 319s, refresh suite 108s, doc audience 3s, decision-key 9s), then STOP on Plan fidelity: the description says the `docs/configuration.md` entry describes batched workers and history growth, and the one-line diff mentions neither. The monitor drafted a "do not post" wait note. Run 3 on the rebased head daaa6660: lint 22s, refresh suite 174s, doc audience 4s, decision-key 8s, then `ship-gate: PASS`. pr-precheck accepted on all five required checks. It still notes that What Changed doesn't list the help rewrite, the `_fm_decision_fold_emit` helper, the `task_json_row` split, or the test helper, and that the evidence counts don't line up.
- Difficulties: the gate is built to run before `gh pr create`, so on an open PR it can only flag description problems I can't fix without editing the PR. Run 1 also called out the Lint block ("linter found issues (exit code 1)") and a title that promises the deadline while the risk note says cost still grows. Both come from the 2026-10-16 pipeline narrative the 2026-11-06 run never refreshed. Each gate run took about 8 minutes, mostly lint.
- What changed: I added `bin/fm-doc-audience-check.sh` to my manifest (exit 0, 141 surfaces). Then I did the PR edit the gate pointed at: I rebased, re-raised through no-mistakes with a clean intent so the description was regenerated, and hand-fixed the `Fixes #6257` link.

---

## Learnings and Reflections

**Technical Skills Gained**

- Profiling Bash by phase instead of by function. Timestamps around the read phase and the composition phase showed the serial composition was the problem, which a per-function profile alone didn't make obvious.
- Removing forks from hot Bash loops with `printf -v` out-variables while keeping the printed form for every other caller, then proving it with byte-identical folds against the base.
- Reading GitHub's CI model for forks: label-gated suites, `action_required` runs waiting for approval, and why a new push restarts both CI and the no-mistakes attestation.

**Challenges Overcome**

- The 2026-11-12 rebase conflict in `_fm_decision_fold_line`. Upstream's new deferral column and my out-variable argument met in the same function header. I merged both sides' locals and proved equivalence with 480 fuzzed folds before pushing. My first fuzz run used the wrong stamp format, so every epoch read `unknown`. I caught that by checking that the folds weren't trivial, and re-ran with real epochs.
- The Keychain lock. Every `gh` call failed with -25308 when my Mac locked, on 2026-10-16 and again on 2026-11-12. Once I recognized the error, the fix was simply unlocking and rerunning, not changing credentials.
- A CI shard I couldn't fix. `Behavior portable serial 2` was cancelled at its 30-minute limit. I proved it runs none of my tests and times out on main too, instead of pushing changes to make it green.
- The resume on 2026-11-06. no-mistakes refused to start because its mirror had diverged. I rebased by hand, resolved the `[timed]` argument conflict, and got the PR back to a clean, reviewed head.
- no-mistakes v1.84.0 stored my credential helper's approval lines as the PR URL. I traced it to how the tool captures `gh pr create` output, and confirmed it only affected display.

**What I'd Do Differently Next Time**

- Read the issue timeline before writing code. Two PRs were already open on #6257 when I opened mine, and one scout grade would have shown me.
- Put `Fixes #<issue>` in the PR description, so the link exists from day one, and check that it renders as a link. A generator put mine in backticks.
- Keep private agent instructions out of the PR body. The Intent section should be written for the maintainer.
- Keep the story file as I go. Reconstructing a month later meant cross-checking author dates against commit dates.

---

## Resources Used

- firstmate `CONTRIBUTING.md`, especially the no-mistakes requirement and "The `full-ci` label": https://github.com/kunchenguid/firstmate/blob/main/CONTRIBUTING.md
- Issue #6257 and its triage comment, which set the 60s target and the "incremental/cache/retention" shape I answered: https://github.com/kunchenguid/firstmate/issues/6257
- no-mistakes quick start, which I followed to raise the PR through the gate: https://kunchenguid.github.io/no-mistakes/start-here/quick-start/
- The `status_open_decisions_incremental` header comment in `bin/fm-classify-lib.sh`, which explained why the cursor path writes state.
