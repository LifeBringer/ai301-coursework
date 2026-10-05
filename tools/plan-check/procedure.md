# Procedure: how this skill grades a plan package

## Read order

1. Decide the mode. A bundle file with `## Repo facts`, `## Issue`, `## Thread highlights`, `## Repro evidence`, `## Candidate plan` and `## Candidate plan comment` is eval mode: the bundle is the only evidence, and `scope.md` and `voice-guide.md` are not read. An issue URL plus the student's `plan.md` and draft comment is live mode.
2. Live mode only: read `scope.md` before anything else. If its `Repo:` line still holds a bracketed placeholder, stop and tell the student to get their cohort's scope file; do not grade. If the issue URL is not in the scoped repo, refuse to grade. Note the house rules it lists.
3. Read `rubric.md` and write down the check names in table order (Grounded diagnosis, Bounded scope, Executable approach, Decisive test plan, Thread and conventions, Calibrated claims), which are required, and the verdict rule. Then read `references/evidence-guide.md`, the map for where each check's evidence lives.
4. Read the package in this order, the whole package before grading anything, and keep notes:
   1. Repo facts (live: the contributing guide, PR template and any AI policy). Copy the contribution-policy and AI-policy wording exactly.
   2. Issue. Note the reported trigger, conditions, visible symptom and expected behavior.
   3. Thread highlights (live: the issue's comments). For each comment by an owner, member, collaborator or maintainer, note any direction about this work. Note every cause anyone proposes, and who proposed it.
   4. Repro evidence (live: the student's own posted reproduction comment, found by its author on the issue). List each step, control, timing and output, then write one line: "the evidence shows X; it rules out or does not reach Y."
   5. Candidate plan (live: `plan.md`).
   6. Candidate plan comment (live: the draft comment).

   Read the evidence before the plan on purpose: a confident, polished plan read first tends to set what the evidence "means". The one-line evidence note from step 4.4 is the reference that Grounded diagnosis and Decisive test plan are measured against.

## Evidence gathering

1. Gather each family from the locations the evidence guide names. In eval mode quote the bundle lines; never fetch or read anything else. In live mode use read-only GitHub access (`gh issue view`, the GitHub API or the web page) and repository files; never edit, post, install or run the student's code.
2. Diagnosis and grounding: copy the plan's cause sentence. For each repro observation from step 4.4 of Read order, mark whether the cause explains it, contradicts it, or leaves it unexplained. Record whether the cause came from the issue or thread, and whether the planned change acts where the cause lives.
3. Scope: list every change the plan commits to, from the scope statement, the named files and each approach step. Tag each one fix, test or docs for the fix, or other. List the explicit out-of-scope or deferred items separately; they are not changes.
4. Executability: for each named location (file, function, component or setting), record the change the plan describes there. Note any step that has no location or no described change.
5. Test plan: list each verification item with its stated expected result. Mark each item that would come out differently on the unfixed and the fixed code, and which repro step or reproduced failing test it re-runs.
6. Comms: list each maintainer direction from step 4.3 of Read order and quote where the comment or plan follows or addresses it, or record that it does not. Quote each policy ask that applies to comments or to the planned work, and quote the comment sentence that meets it, or record its absence. In live mode also check the house rules from `scope.md`.
7. Honesty: record the stated risks and unknowns, certainty words, promises and timelines in the comment, and the content of any Deviations section.
8. Record evidence once and reuse it. If a family's evidence is genuinely absent from every location the guide names, record "absent" with the locations read.

## Check execution

1. Grade the checks in rubric table order. Diagnosis comes first because Bounded scope and Decisive test plan are both read against the cause it settles.
2. For each check, apply only that row's pass condition to the evidence recorded for it, and grade it `pass`, `fail` or `unclear`. Give one line naming the fact or quote that decided it.
3. When the plan or comment leaves out something the pass condition requires it to state (no cause, no location, no expected fixed result, no required disclosure), grade `fail`. Use `unclear` only when the package lacks a source the check needs and the candidate could not have supplied it, for example a missing repro-evidence block.
4. Grade each check on its own evidence. A failure in one check never lowers another check's grade, and a strong check never rescues a weak one.
5. Grade substance, not presentation. Length, headings, polish and confident tone are not evidence; a terse plan can pass every check and a long one can fail.
6. Re-read the package only for a check whose evidence was not recorded during gathering. When the evidence supports both a pass and a fail reading, decide by the literal pass condition and mention the tension in the summary.

## Verdict assembly

1. Take the five required grades: Grounded diagnosis, Bounded scope, Executable approach, Decisive test plan, Thread and conventions.
2. If all five are `pass`, the verdict is `accept`. If any is `fail` or `unclear`, the verdict is `reject`. There is no other verdict.
3. Calibrated claims is preferred: include its grade and evidence, but never let it change the verdict.
4. Write a short summary with one line per check. For a reject, name the first failing or unclear required check in table order and quote its deciding evidence. For an accept, name the check that came closest to failing and why it still passed.
5. Live mode only: compare the draft comment with each rule in `voice-guide.md` and quote any rule it breaks. Voice findings never change the verdict. Also report any step of this procedure that did not fit the package, rather than improvising around it.
6. End with the fenced JSON block from `SKILL.md`: the item id (bundle id or issue URL), all six checks in table order using the rubric's check names, and the verdict. Nothing follows the JSON block.
