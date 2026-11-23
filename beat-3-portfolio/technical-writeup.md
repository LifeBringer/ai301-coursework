# Keeping firstmate's home summary inside its 60-second deadline: a merged fix, and the gate that stopped me twice

## Overview

[firstmate](https://github.com/kunchenguid/firstmate) is an open source tool that runs a crew of AI coding agents. Its home summary, a JSON file with the state of every task, must refresh inside a 60-second deadline. On homes with many tasks and long status histories, the refresh ran past the deadline and was killed, so the summary stopped updating ([#6257](https://github.com/kunchenguid/firstmate/issues/6257)). I hit this on my own machine and wrote [PR #7376](https://github.com/kunchenguid/firstmate/pull/7376). It builds each task's summary row in that task's own parallel worker, and it stops the status-history fold from forking subshells for every line. It merged on 2026-11-12 as `ed7fbb84`, and #6257 closed. Before my last push, I ran the course's ship-gate with the two parts I wrote: the manifest of four repo checks from CONTRIBUTING.md, and my unit 4 pr-precheck rubric. It stopped me twice, and both stops changed what I shipped. An AI assistant helped me write the change and run the checks, and the PR says so.

## What happened

### 1. I hit the bug before I found the issue (2026-10-14)

My own firstmate home logged 118 `refresh exceeded its 60-second deadline` failures between 2026-09-30 and 2026-10-14, and 82 of them were on 2026-10-14. I reproduced the failure directly (exit 124), then moved all further work into disposable lab homes.

The issue suggested a cursor-based incremental fold, so a cursor looked like the obvious fix. A phase timing of the failing run told a different story. The bounded reads finished at 23.4s. Row composition started at 23.5s and ran one task at a time. Only 6 of 12 rows were done when the deadline hit, and the largest task's fold alone ran from 36.2s to 57.7s. Each row re-folded the task's whole status history through `status_open_decisions`, and each decision transition forked four to five subshells. My largest log needed about 1,300 forks.

### 2. The fix (2026-10-14 to 2026-10-16)

I made two changes. First, each task's fold and row now run inside that task's observation worker. They overlap the bounded read and run in parallel across tasks, still in batches of `FM_SNAPSHOT_LOCAL_READ_CONCURRENCY` (default 8). From `bin/fm-fleet-snapshot.sh`:

```diff
+  # The whole-stream open-decision fold costs time proportional to the task's
+  # status history, so it runs here while the bounded current-state read is
+  # still outstanding rather than after every read has finished.
+  status_open_decisions "$status_capture" "$kind" > "$open_file" || current_rc=1
   [ -z "$current_pid" ] || wait "$current_pid" || current_rc=1
@@ ... @@
-  return "$current_rc"
+  [ "$current_rc" -eq 0 ] || return "$current_rc"
+  task_json_row "$meta" "$id" > "$SNAPSHOT_TASK_DIR/$id.row"
```

Second, the fold helpers in `bin/fm-classify-lib.sh` take an optional output variable and fill it with `printf -v`, so the whole-file fold starts no subshell per line:

```diff
-        open=$(_fm_decision_fold_line "$open" "$line" "$resolve" "$held" "$kind")
+        _fm_decision_fold_line "$open" "$line" "$resolve" "$held" "$kind" '' folded
+        open=$folded
```

Without the variable, each helper prints exactly what it printed before, so every other caller is unchanged. I kept the whole-stream fold and did not use the cursor. The existing cursor path writes state as a side effect, and the snapshot is a read path with its own test that it changes no state.

firstmate requires outside PRs to go through the no-mistakes pipeline. Its review found that my row temp file name, `$id.row.json`, could collide for task ids `a` and `a.row`, and it renamed the file to `$id.row`. I added three test cases: a refresh killed by the deadline keeps the last summary, four task rows compose at the same time, and a 12-task fleet with hundreds of decision transitions per task publishes inside the default deadline. The pipeline opened the PR on 2026-10-16.

### 3. Approved, then stalled (2026-11-06 to 2026-11-12)

I parked the work for three weeks, then rebased onto main on 2026-11-06. The maintainer's agent reviewed the new head the same day:

> I've added the `full-ci` label so the full suite runs; if it's green this merges on its own. Nothing more is needed from you.

It did not merge. The full-suite run waited at `action_required` for fork-workflow approval, and main kept moving. On 2026-11-12 I ran my Beat 2 tools on the open PR. My PR monitor caught a problem I had not seen (from `monitor-log.md`):

```text
2026-11-12 09:22 [state] https://github.com/kunchenguid/firstmate/pull/7376
- state: feedback-in: review feedback is in (latest from the thread:
  comment 5 days ago)
- mergeability: CONFLICTING/DIRTY
```

My issue scout caught a worse one. It rejected #6257 on its Available work check: "timeline cross-references open PRs #6464 (luigia, 2026-10-03) and #6980 (zico-lle, 2026-10-10)". Two fixes were open before mine. I had never read the issue timeline, and my PR description never named #6257, so nothing connected them.

### 4. A reproduction anyone can rerun (2026-11-12)

I built a public reproduction from the PR's own test file: 12 tasks with about 508 status lines each, and the default 60s deadline. I ran it on current main and on my head, on a quiet machine and under contention (background QoS plus 4 busy loops). These are the results from the comment I posted on #6257:

| Code | Quiet | Contended |
|---|---|---|
| main `a9ae3c0c` | 1 run: `INCIDENT_WALL=37s`, ok | 1 run: `not ok - the incident-shaped fleet did not publish within the default deadline` |
| #7376 head `fd404cbd` | 1 run: `INCIDENT_WALL=6s`, ok | Run 1 stopped early at an existing case. Run 2: `INCIDENT_WALL=20s`, ok |

The same comment named both earlier PRs and ended: "Please let me know which direction you want to keep, and I'll close mine if it isn't this one."

### 5. The ship-gate stopped me twice (2026-11-12)

The ship-gate checks a branch before it ships. It runs every check in my manifest, then grades the branch with my pr-precheck rubric, and it says PASS or STOP. My PR already existed, so I ran `check_gate.py --branch` by hand. Run 1 said STOP:

```text
ship-gate: STOP. your own pr-precheck rejected
  fm/firstmate-home-summary-timeout: verdict reject.
  - Plan fidelity [unclear]: Gate diffed against a stale local main (516 files,
    99,908 insertions, hundreds of upstream commits) and truncated at 120,000
    chars before bin/ or tests/, ...
  - Repo checks reported [fail]: CONTRIBUTING says to run
    bin/fm-doc-audience-check.sh after documentation changes; the PR edits
    docs/configuration.md and no outcome for that command appears; ...
```

Two of these findings were my mistakes as the tool's author, not problems in the branch. The gate diffed against my stale local main, so I passed `--base upstream/main`. My manifest also skipped a check that CONTRIBUTING.md requires. Run 1 ran three checks:

```text
ship-gate: repo check 1/3 `Lint (ShellCheck 0.11.0 + actionlint 1.7.12)`:
  bin/fm-lint.sh
ship-gate: repo check 2/3 `Home-summary refresh suite`:
  bin/fm-test-run.sh tests/fm-home-summary-refresh.test.sh
ship-gate: repo check 3/3 `Decision-key fold suite`:
  bin/fm-test-run.sh tests/fm-classify-decision-key.test.sh
```

I added this row to the manifest's "Must pass before a ship" table:

| Check | Command | Source |
|---|---|---|
| Documentation audience check | bin/fm-doc-audience-check.sh | CONTRIBUTING.md, Repo conventions: "run bin/fm-doc-audience-check.sh after documentation changes" (the PR edits docs/configuration.md; added 2026-11-12 after the first gate run) |

Run 2 passed all four checks (`Documentation audience check: ok (3s)`) and still said STOP, on one row:

```text
  - Plan fidelity [fail]: Description says the --help text and the
    docs/configuration.md entry 'describe the batched workers and note that
    collection time still grows with accumulated status history'; the docs
    diff is one line adding the fold and row composition to the concurrency
    cap, with no mention of batches or history growth.
```

The gate was right. My description claimed more than my one-line docs change did. To fix it, I had to update the PR, and the branch also conflicted with main, so I fixed both together. I rebased onto `a9ae3c0c` and resolved the conflict in `_fm_decision_fold_line`, where upstream had added a deferral column next to my output-variable argument. Then I checked that the fold still matched main: 480 of 480 fuzzed fold comparisons were equal byte for byte. I raised the branch through no-mistakes again, and the pipeline wrote a new description. The no-mistakes test step measured the real refresh at 7s on the new head and 39s on main. The docs claim changed:

- Before: the `--help` text and the `docs/configuration.md` entry "describe the batched workers and note that collection time still grows with accumulated status history"
- After: "`docs/configuration.md` updates the `FM_SNAPSHOT_LOCAL_READ_CONCURRENCY` description to match the new behavior."

The generator put `Fixes #6257` in backticks, which breaks the issue link, so I made one hand edit. Run 3, on the new head `daaa6660`:

```text
ship-gate: PASS. fm/firstmate-home-summary-timeout passed your manifest's checks
  and your pr-precheck.
  Plan fidelity [pass]: Every hunk maps to a plan item ... description claims
    match the diff.
  Template and policy asks [pass]: No PR template; 'Fixes #6257' present; ...
  Calibrated claims [fail]: '480 ... folds over 40 fuzzed status logs' is not
    shown (pipeline evidence shows 80 logs, seeds 1 and 2); ...
```

Calibrated claims is a preferred check, so it does not change the verdict. I come back to it below.

### 6. Merged (2026-11-12)

That afternoon, the maintainer's agent read the full diff of `daaa6660` and approved the fork CI runs:

> Contract class is still **restore**: main already promises the 60-second home-summary refresh, and this change makes that path finish in time without changing the deadline, bounds, or published content.

The full suite passed, and the PR merged at 23:46 UTC as `ed7fbb84`. GitHub closed #6257 as completed. Less than four hours later, the maintainer closed both earlier PRs as superseded. The note on #6464 says:

> #6257 was fixed by #7376, which landed on main today as `ed7fbb84`.

## Limitations and what's next

The fix keeps the refresh inside its deadline today, but it does not put a limit on the cost. The fold still reads each task's whole status history on every refresh, at about 2ms per line on bash 3.2. If the histories keep growing, the refresh will reach 60s again. The maintainer said the same thing when closing #6464:

> One possible case: #7376 notes that collection cost still grows with status-log history, which is what your cursor-seeded tail fold targets. If you can show that cost on current main, I'll look again.

Next, I want to measure that limit on current main. I will grow the incident home's histories in steps and find the line count where the refresh crosses 60s, quiet and contended. That number decides if firstmate needs an incremental fold on the snapshot path. If it does, the fold must be read-only, because the snapshot must not write state. All my timings also come from macOS with stock bash 3.2. The report came from Linux with bash 5, and I did not time that setup.

The second limit is in how I used the gate. The gate is built to check a branch before `gh pr create`, but I ran it four weeks after my PR opened. So the overstated docs claim and the missing issue link were in public for those four weeks. The merged description still has the claim that failed the gate's last run: it cites 480 fold comparisons that the attached evidence does not show. Next time, I will register the gate hook so it runs before the PR opens. I will also grade the issue timeline with my scout before I write any code, so I find earlier PRs before I start, not after.
