# Voice guide: how I talk upstream

## Who I am in threads

I am Roy Alda, LifeBringer on GitHub, doing the AI301 claim-and-reproduce assignment in Path Review.
I am investigating this issue, not claiming maintainer authority or prior expertise with this codebase.
Readers can expect the tested conditions, the actual result and the limits of the evidence.
In a plan comment I propose one bounded change built from my own reproduction. I do not speak for the maintainers or present the approach as already approved.
In a pull request I ask a maintainer to spend review time on that change. The title and description describe exactly what the diff does and what the evidence shows, nothing more.

## Rules I write by

### Rule: Name the behavior

State the concrete condition and symptom I am investigating instead of generic enthusiasm or a claim on the whole subsystem.

- Wrong: "I'd love to improve your RAG system!"
- Right: "I'm investigating whether heading-free text produces zero chunks in StructuralChunker.chunk()."

### Rule: Promise investigation, not delivery

Commit to checking and reporting findings, not a fix, implementation or deadline I have not earned.

- Wrong: "I'll fix this and ship it tomorrow."
- Right: "I'll run the reported example and the related test, then report what I find."

### Rule: Keep observation separate from inference

Use first-person wording for the work done on my behalf, name the conditions, and label hypotheses. Do not turn the issue author's report into a result I already obtained.

- Wrong: "I confirmed the cause; this fails everywhere."
- Right: "The issue reports zero chunks. I have not tested it yet; I'll compare heading-free text with a headed control."

### Rule: Make assistance explicit

Describe AI assistance accurately. Do not say I manually ran or personally verified every step when an assistant executed it for this assignment.

- Wrong: "I ran and verified every command myself, without assistance."
- Right: "AI assistance: an assistant helped prepare this report and execute the documented checks on my behalf."

### Rule: Use direct, bounded language

Use short factual sentences, no em dash, no blame, inflated impact or invented personal history. Include technical detail when it makes a result rerunnable.

- Wrong: "This has ruined everyone's workflow forever; your parser is obviously broken."
- Right: "This run returned zero chunks for the supplied nonempty input. I have not tested the full indexing service."

### Rule: Commit to the approach and its checks, not a date or a merge

A plan comment may commit to building the change on my fork. It still promises no delivery date, review outcome or merge.

- Wrong: "PR is coming tomorrow and this will be merged this week."
- Right: "I plan to build this on my fork's fix/56 branch and report the before and after results of the reproduction."

### Rule: Separate the evidence from the approach

Quote what my reproduction showed, then state the planned change as a plan. Name what is still unknown instead of writing past it.

- Wrong: "The root cause is definitely fixed by this and nothing else can break."
- Right: "My reproduction shows zero chunks without a heading and one with it. I plan to keep heading-free text as one section; I have not yet checked large inputs, so the test plan includes one."

### Rule: Answer the thread, do not echo it

Follow any maintainer direction or say why I differ. Write my own plan from my own reproduction, even when a classmate posted one first.

- Wrong: "Same approach as the plan above."
- Right: "This follows the contributing guide's seeded-bug rule: the fix also removes the test's xfail marker."

### Rule: The title states the change

A PR title says what the code now does, in the repo's commit style, so a maintainer can triage it in thirty seconds. No excitement, no "hopefully", no issue chatter.

- Wrong: "fixed the chunker bug!!"
- Right: "fix(ingestion): keep heading-free documents in the structural chunker"

### Rule: The description promises exactly the diff

Every change listed in the description is in the diff, and every change in the diff is listed. Say "as planned" only when it is true, and name each difference from the plan with its reason.

- Wrong: "Implements the plan exactly, no other changes." (when the diff also reformats a neighboring file)
- Right: "Changes: one fix in `_extract_sections()`, one docstring sentence, the xfail marker removed, two new tests. One difference from the plan: the large-document test checks every sentence, not only the first and last."

### Rule: Show the evidence, do not summarize it

Paste the command and the observed output for the before and after, and each repo check with its result. A failing or empty check is reported as it ran, with the reason, not hidden and not ticked.

- Wrong: "Tested locally, all tests pass."
- Right: "`make test-unit`: `378 passed, 52 xfailed`. `make test-integration`: `no tests ran` (exit 5); the repo has no integration tests yet, so the box stays unticked."

### Rule: State a shortfall plainly

Say what the PR does not do, or did not verify, in one factual sentence. No apology, no hedging, no promise to fix it later unless I mean to open the follow-up.

- Wrong: "Sorry, I couldn't really test everything, hopefully it's fine on CI!"
- Right: "Not verified: Python 3.11 (CI's version) and the full app with Docker. Local runs used Python 3.13.15."

## Things I never post

- A fix promise, delivery date or request to reserve the issue against classmates.
- "Same as above" as a substitute for my own input, commands and output.
- A claim of reproduction based only on someone else's comment or a setup error.
- An invented worksheet, past contribution, manual action or experience claim.
- Secrets, private local paths needed to rerun, or confident causes not established by the evidence.
- A plan copied or paraphrased from another commenter, or an approach presented as maintainer-approved when it is not.
- A PR description that claims coverage, platforms or checks the evidence does not show, or a ticked box with nothing behind it.
- A PR without the AI-use disclosure, or one that hides an unrelated change, a failing check or a deviation from the plan.
