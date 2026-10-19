# Unit 4 - Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

---

## Your pull request

**Pull request**

https://github.com/codepath/pathreview-ai301-fa26-s1/pull/227

**Branch**

`fix/56-keep-heading-free-docs`

## Eval iterations

**Run history**

Every run used the unchanged official harness from `codepath/ai301-unit4-starter` at `76e4b7382740da2a496e538ae2a0ece3f07272fd`, its pinned Sonnet grader, and the installed files in `~/.claude/skills/pr-precheck/`. In order, on 2026-10-19:

1. Full run, v1: `agreement: 17/20 scored items  (bar: 18/20: below the bar)`. `categories: clear-accept 4/7  not-tested 4/4  silent-drift 4/4  standards-wall 2/2  unreviewable 3/3`. Zero errors. All three disagreements were clear accepts that I rejected:
   - pkg-05 and pkg-13 on Decisive test evidence. Each re-ran its reproduced failure with real output but only stated a control or a new-behavior check in prose.
   - pkg-08 on Repo checks reported. Its "code formatted" checklist answer had no formatter output behind it.

   I revised two checks. Decisive test evidence now needs a shown run only for each reproduced failure. Repo checks reported now covers commands to run, and checklist attestations moved to Template and policy asks. The procedure and evidence guide changed to match, and SKILL.md got a working `gh issue view` command for live mode.
2. Targeted retry, `--only pkg-05,pkg-08,pkg-13,pkg-04,pkg-07,pkg-01`: `agreement: 6/6 scored items`. The three clear accepts now matched. The canaries pkg-04 and pkg-07 (not-tested) and pkg-01 (standards-wall) all stayed reject. This partial run did not write `eval-run.txt`.
3. Full run, v2, with `--save-run eval-run.txt`: `agreement: 20/20 scored items  (bar: 18/20: PASS)`. `categories: clear-accept 7/7  not-tested 4/4  silent-drift 4/4  standards-wall 2/2  unreviewable 3/3`. Zero errors. This is the committed `eval-run.txt`, and no skill file changed after it. Its header fingerprints match the uploaded files.

The live runs on my own PR were not eval runs. Live run 1 (v1) rejected my draft on Decisive test evidence, because my plan's test-plan step 5 (an edge-case probe) was never re-run in `test_evidence.md`. I re-ran it before and after and added it to the evidence and the description. After that, live run 2 (v1) and live run 3 (v2, the final files) both returned `accept` on all six checks.

**Package analysis**

pkg-13 (`neovim/neovim#41337`, category clear-accept). In the final run my rubric decided `accept`, and the gold label is `accept`:

```text
pkg-13  clear-accept    accept  accept   yes
```

The plan names a single bounded change: caret-escape the cmd.exe metacharacters in `vim.ui.open()`. Its test plan has three items: re-run the repro and expect "the full URL (both query parameters) in the browser and exit code 0", keep "the no-ampersand control unchanged", and run the ui functional spec. The PR's evidence re-runs the reproduced failure through the input that failed: `-- before: { code = 1, stderr = "'t' is not recognized ..." }` and `-- after:  { code = 0, stderr = "" }`, with the browser showing `?q=hello&t=brave`. That is the plan's expected fixed result, so Decisive test evidence passes. `TEST_FILE=test/functional/lua/ui_spec.lua make functionaltest` "passes, including the new case" is a named check with its outcome, so Repo checks reported passes. The diff is the fix in `ui.lua` and its test, which passes Plan fidelity and Reviewable diff. The description restates the plan's known limit: "percent-sign environment expansion (`%VAR%`) is NOT handled here". That is an honest shortfall, not drift.

My v1 rubric rejected this package. v1 asked for a shown run of every test-plan item, and the evidence only says "No-ampersand control unchanged." The grader's v1 evidence was: "'No-ampersand control unchanged' is asserted in prose with no command or observed output". But a control never failed, so a re-run of it proves nothing about the fix. The failure the issue reported is the one that must be shown. I agree with the gold label: the bug is re-run before and after, the limit is disclosed, and the diff is clean.

**Check rationale**

The exact current Decisive test evidence row in `tools/pr-precheck/rubric.md`:

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| Decisive test evidence | The test evidence and anything the description pastes, read against the plan's test plan and the reproduction evidence it re-runs (Test evidence). | For every reproduced failure the plan's test plan says to re-run (each failure mode, input or failing test the repro evidence showed failing), the evidence shows the run on the changed code (a command or test name with its observed after-output), and that output is the plan's expected fixed result or visibly differs from the reproduced before-result. The run must go through the path that failed: a control, a different input, or an unchanged path that never broke does not stand in for it. A reproduced failure that was not re-run passes only if the PR says so and why. Other test-plan items (controls, regression or edge cases, checks of new behavior the plan adds, suite runs) need only be named with a stated outcome; they do not need pasted output. Prose such as "tested locally", "works now" or "tests pass" in place of the reproduced failure's re-run does not count. | required |

It is built around the reproduced failure because that is the only evidence that changes between the unfixed and the fixed code. The not-tested category fails PRs whose evidence never touches the broken path: "tested locally", "tests pass", or a control that never broke. So the check asks for the failing input's own re-run, with observed output, and names the controls and unchanged paths that cannot stand in for it. "For every reproduced failure" is there because a plan that reproduced two failure modes needs two re-runs; the class calibration package calib-04 re-ran only one of them.

The row is my v2. In v1 the first sentence read "For every reproduced failure the plan's test plan says to re-run (each failure mode, input or named failing test it lists)". In practice that meant every test-plan item, and the grader demanded pasted output for controls and new-behavior checks too. That rejected three clear accepts (pkg-05, pkg-13, and with the matching Repo checks wording, pkg-08). I added the definition "each failure mode, input or failing test the repro evidence showed failing", and the sentence that lets other items be "named with a stated outcome". I rejected dropping the shown-run requirement altogether, because pkg-04's only evidence, "Tested locally on Windows 11 with less-Windows: colors work now in paging mode", would then pass.

**Trade-offs**

The v2 wording gives up proof for anything that never failed. A PR can now write "control unchanged" or "the new warning prints" without pasting a run, and pass, even if that sentence is wrong. I accept that miss. A false control claim is a Calibrated claims problem for a reviewer, and the decisive evidence, the reproduced failure's before and after, is still required.

To check that the loosening did not let untested PRs through, I re-ran `--only pkg-05,pkg-08,pkg-13,pkg-04,pkg-07,pkg-01`:
- pkg-04 and pkg-07 are not-tested packages that v1 had failed on no required check except the two I loosened, so they were the most exposed. Both stayed reject on Decisive test evidence: pkg-04 shows no command or output for its paged-color repro, and pkg-07 re-runs only the single-file control ("Verified the fix on a single-file case").
- pkg-01 is a standards-wall canary for the checklist items that moved to Template and policy asks. It stayed reject.

The confirming full run then kept every v1 agreement and reached 20/20.

---

Related paths: `eval-run.txt` in this directory; the skill's files in `tools/pr-precheck/`.
