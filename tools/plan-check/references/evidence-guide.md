# Evidence guide: where evidence lives in a plan package

An eval bundle has six parts, in this order: `## Repo facts`, `## Issue`, `## Thread highlights`, `## Repro evidence`, `## Candidate plan` and `## Candidate plan comment`. In live mode the same evidence comes from the issue page and its comments on GitHub, the repository's own docs, the student's posted reproduction comment on that issue, and the two drafts (`plan.md` and the draft comment). The drafts are the package: other files in the student's working directory, and other commenters' reproductions, are not evidence for it.

## Diagnosis and grounding

Where it lives:

- Eval: the cause sentence or diagnosis section of `## Candidate plan` (often under "Cause", "Diagnosis" or "Root cause", sometimes the first line of the plan), read against `## Repro evidence`: its numbered steps, controls ("Control:", "without X"), timing tables, observed outputs and the "Actual" line. Causes proposed by others appear in `## Issue` and `## Thread highlights`.
- Live: the diagnosis in `plan.md`, read against the student's own posted reproduction comment on the issue and the evidence quoted in the drafts. Commenters' proposed causes are in the issue body and thread.

What good looks like: the cause names a mechanism, and every repro observation fits it, including the control or condition that makes the symptom appear or disappear. Test it by asking what the cause predicts for each step: if the repro shows the symptom with the suspected component removed (for example the delay still happens with no pager in the loop), or shows it vanish when an unrelated condition changes, the cause is contradicted. A cause adopted from the thread earns no credit for being confident or popular; it must fit this package's evidence. The change must act where the cause lives, not add a retry, a catch-all or a cosmetic refresh over a symptom whose cause is elsewhere.

## Scope

Where it lives:

- Eval: in-scope and out-of-scope statements ("In:", "Out:", "Not in scope", "Scope"), the files or areas named, and each numbered approach or change step in `## Candidate plan`; the summary of the change in `## Candidate plan comment`. The reported behavior it must stay tied to is in `## Issue` and `## Repro evidence`.
- Live: the scope, files and approach sections of `plan.md` and the summary in the draft comment, read against the issue body and the student's posted reproduction.

What good looks like: list every change the plan commits to and tag each one fix, test or docs for the fix, or other. A bounded plan has no "other": no rename, cleanup, dependency bump, new option, schema change or rewrite of neighboring behavior that the reproduced failure does not need. Items the plan explicitly excludes or defers to a separate issue are evidence of discipline, not creep. Several files can be one bounded change when each edit serves the same fix.

## Executability

Where it lives:

- Eval: the files, functions, components or settings named in `## Candidate plan`, its approach steps and their order.
- Live: the files and approach sections of `plan.md`.

What good looks like: a stranger who knows the repository could open the named location and start editing without asking the author anything, because the plan says where the change goes and what the code will do differently there (for example "add the commits context to the post-push refresh scope in `sync_controller.go`"). "Poke around the editor", "figure out where it lives" or "improve the handling" names neither.

## Test plan

Where it lives:

- Eval: the "Test", "Test plan" or "Verification" part of `## Candidate plan`, and any verification promised in `## Candidate plan comment`, read against the steps, inputs and observed outputs in `## Repro evidence`.
- Live: the test plan section of `plan.md` and the draft comment, read against the commands and outputs in the student's posted reproduction.

What good looks like: at least one check that would come out differently on the unfixed and the fixed code, with the expected fixed result stated: the repro step re-run with what it should now show (for example "at step 3 the color flips without leaving the view"), a named failing test that should now pass on its real assertion, or a new test whose assertion encodes the reproduced behavior. A whole-suite run or "no regressions" is a useful extra, but on its own it observes nothing about this fix.

## Honesty

Where it lives:

- Eval: risks, unknowns, assumptions and caveats in `## Candidate plan`; certainty words ("confirmed", "root cause proven", "guaranteed") and promises or timelines in `## Candidate plan comment`; the limits stated in `## Repro evidence`.
- Live: the risks and unknowns section and the final `## Deviations` section of `plan.md`, and the draft comment.

What good looks like: what the repro did not establish stays labelled as unknown or hypothesis, what the plan says others said in the issue or thread matches who actually said it, and the comment promises an approach and its checks, not a merge date or an outcome the test plan cannot observe. After a build, Deviations records what changed from the posted plan and why, or says plainly that nothing changed. A deviation that exists only in the diff is not recorded.

## Comms

Where it lives:

- Eval: `## Thread highlights`, where each entry carries the commenter's role (OWNER, MEMBER, COLLABORATOR or maintainer versus NONE or CONTRIBUTOR); the contribution-policy and AI-policy line in `## Repo facts`; and `## Candidate plan comment`, the words that would be posted.
- Live: the issue's comments with their author association (`gh issue view <number> -R <owner/repo> --comments`, or the API's `author_association` field); the repository's contributing guide (Path Review keeps it at `docs/CONTRIBUTING.md`, linked from the README), PR template and any AI policy; the house rules in `scope.md`; and the draft comment.

What good looks like: for each maintainer statement that gives direction on this work, the comment either follows it or names it and explains the difference so the maintainer can decide; non-maintainer claims are context, not direction. Judge what the comment does about each direction; how precisely the plan paraphrases or attributes thread remarks is an Honesty question. For each policy ask that applies to comments or to the planned work, the comment meets it in the stated form. In particular, an AI-use disclosure requirement is met only by a disclosure in the comment, never by silence, because course work is AI-assisted. Repo conventions that govern the planned change itself (for example Path Review's rule that fixing a seeded bug also removes its `xfail` marker) are met when the plan includes them. A comment that restates the plan in its own words, built from the student's own reproduction, is thread-aware; "same approach as above" is piggybacking (a Path Review house rule in live mode).
