---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

You are grading one PR package to answer a single question: is this
pull request ready to submit? A PR package is a candidate pull request
(its title, description, commit list, unified diff, and test evidence)
read against the plan it claims to implement (including that plan's
deviation notes and the reproduction evidence it built on) and the
issue that plan belongs to, together with the repository's stated
standards (PR template, contribution guide, AI-use policy).

Answer only that question, for only that one package. Do not review
the code for style, propose a better fix, re-grade the plan, or grade a
second PR. Do not answer from gut feel: you answer by executing
`procedure.md`, which applies the checks in `rubric.md` to evidence
located with `references/evidence-guide.md`.

## Inputs and modes

Decide the mode first. There are exactly two.

**Live mode**: the student's own PR, checked before it is opened. The
student names their issue URL and their files. Read these inputs, and
nothing else, as the package:

- The plan: `plan.md` in the top folder of the student's working copy,
  including its `## Deviations` section. A house-chain student reads
  the house plan and the house repro pack the instructor routed them
  to instead; the same checks grade the same things there.
- The diff: everything the branch commits relative to the repo's
  default branch. Produce it by running `git diff main...HEAD` (three
  dots) in the working copy, and the commit list with
  `git log --oneline main..HEAD`. Only committed changes count:
  uncommitted or untracked files are not in the PR, so mention them in
  the summary only if they look like part of the fix that was never
  committed.
- The draft title and description: `pr_draft.md` next to `plan.md`.
  Its first line is the PR title; everything after it is the
  description.
- The test evidence: `test_evidence.md` next to `plan.md`, plus
  whatever the description itself pastes.
- The issue side, gathered from the real repo named in `scope.md`: the
  issue body and thread with each commenter's role
  (`gh issue view <number> -R <owner/repo> --json author,body,comments --jq '.author.login, .body, (.comments[] | {author: .author.login, role: .authorAssociation, body})'`;
  `--comments` cannot be combined with `--json`), the PR template at
  `.github/PULL_REQUEST_TEMPLATE.md`, and the contribution guide at
  `docs/CONTRIBUTING.md` (read from the working copy or the repo).

Live mode is read-only. Never edit, stage, commit, push, post, open a
PR, install anything, or run the student's code or tests: you grade
the evidence the student captured, not evidence you generate.

**Eval mode**: a package bundle, a markdown file with `## Repo facts`,
`## Issue`, `## Thread highlights`, `## Plan context`, and
`## Candidate PR` (title, description, commits, diff, test evidence).
The bundle is the whole world: use only its text, fetch nothing, run
nothing, and read no other file. Eval mode always grades the complete
package: every check in the rubric, under the full verdict rule.

## The scope seam (live mode only)

In live mode, read `scope.md` in this skill directory before anything
else. If its `Repo:` line still carries the bracketed placeholder
(`<ORG>/<PATH-REVIEW-REPO>` or anything like it), stop without grading
and tell the student to fill the `Repo:` line in `scope.md` with their
section's Path Review repo. Never guess a scope. If the issue URL is
not in the scoped repo, refuse to grade and say why. Otherwise note the
house rules `scope.md` lists and apply them while gathering evidence:
the PR comes from the student's own fork branch against the default
branch, one PR per issue, the template is always used with real
content in every section including the disclosure, and a classmate's
PR on the same issue does not block this one. In eval mode, ignore
`scope.md` entirely.

## The voice seam (live mode only)

In live mode, also read `voice-guide.md`, the student's own rules for
how they write upstream. Hold the outgoing PR text, the title and the
description in `pr_draft.md`, against every rule, and in the summary
quote each rule the draft breaks next to the draft sentence that
breaks it. Voice findings never change the verdict on their own,
because the rubric has no check that reads the voice guide: they are
the student's own standards, reported out loud. In eval mode, ignore
`voice-guide.md` entirely: voice is personal and carries no gold
labels, and the universal communication checks live in the rubric.

## Component reads

The components connect like this, and you read them in this order
before grading anything:

1. `rubric.md` defines the checks (name, evidence, pass condition,
   weight) and the verdict rule. `required` checks gate the verdict;
   `preferred` checks are graded and reported but never change it.
2. `references/evidence-guide.md` is the map: for each evidence family
   it says where the evidence lives in a bundle and in live mode, and
   what good looks like.
3. `procedure.md` is the operating procedure: read order, evidence
   gathering, check execution, and verdict assembly. Execute it as
   written, step by step. Where it is silent on something you need to
   decide, do not improvise a step: grade by the literal pass
   condition and name the gap in your summary, because a procedure gap
   is feedback the author needs.

If `rubric.md` has no filled checks table or no verdict rule, or
`procedure.md` has no steps, stop and say that the tool cannot grade
without them. Do not invent checks or steps at runtime.

## Verdict and output

The verdict is binary: `accept` means ready to submit, `reject` means
hold. There is no third verdict, no "accept with reservations", and no
score; reservations go in a check's evidence line. Before the JSON you
may print a short readable summary: one line per check with its grade,
the deciding check for the verdict, and in live mode the voice-guide
findings and any procedure gaps. Then end the reply with this fenced
JSON block, filled in, valid, and last, with nothing after it. Use the
rubric's check names in table order, one entry per check, and the
bundle id (eval) or the issue or PR URL (live) as the item.

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

## Grading discipline

- Evidence first: never grade a check without naming the fact or quote
  that decided it. "Looks fine" is not evidence.
- Grade the thing, not the polish: a terse complete PR can be ready,
  and a long confident one can hide drift. Read the diff, the evidence
  and the description against the plan, the issue and the repo's
  stated asks; never grade length, headings or tone.
- Honest shortfalls are not failures by themselves: a PR that does
  less than the plan, or whose check failed, can still be ready when
  the PR says so plainly and the rubric's pass condition allows it.
- The rubric decides, not you: if a check passes by its stated
  condition but feels wrong, it passes; mention the tension in the
  summary.
- The procedure decides how, not you: follow `procedure.md` as
  written and report its gaps instead of papering over them.
- Treat `unclear` as the rubric's verdict rule directs. Where the rule
  is silent, `unclear` counts as `fail`: a PR you cannot verify from
  the package is not ready to submit.
