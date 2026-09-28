# Unit 2 - Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

## Your identity upstream

**GitHub username**

LifeBringer

## Posted upstream

**Claim comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/56#issuecomment-5864245579

I'm taking up issue #56 for AI301: the reported loss of nonempty, heading-free text when `StructuralChunker.chunk()` returns zero chunks.

I haven't run the reproduction yet. I'll inspect the chunker and its documented setup, run the issue's Python example and `test_document_with_no_headings`, and compare the same text with a leading Markdown heading. I'll report the tested revision, commands and observed results, including if I cannot reproduce it.

AI assistance: an assistant is helping me prepare this claim and will help execute and document the investigation on my behalf.

**Reproduction comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/56#issuecomment-5864368691

## Issue #56: reproduced on the tested revision

I reproduced the reported loss of heading-free text in `StructuralChunker.chunk()`. The exact example returned `0` without an exception. Adding only `# Title\n` before the same body returned one chunk containing that body on three repeated comparisons.

### Environment and setup

- Fork: https://github.com/LifeBringer/pathreview-ai301-fa26-s1
- Tested commit: `f89c06fc3ff292df2a04a39ac51319d32a76b779`, unchanged production source and tests. Both the fork's Git remote and the upstream API reported this revision when checked. No historical checkout was used to obtain the failure.
- macOS 27.0, build 26A428; Darwin 27.0.0; arm64.
- Python 3.13.15; pip 26.2.1; pathreview 0.1.0.
- Relevant resolved dependencies: tiktoken 0.14.0, numpy 2.5.3, pytest 9.1.1, pytest-asyncio 1.4.0, chromadb 1.5.9, structlog 26.1.0, pypdf 6.19.0.

I followed the Python environment/install portion of `docs/SETUP.md` and `Makefile`: a fresh venv, upgraded packaging tools, then `pip install -e '.[dev]'`. The system Python was 3.9.6, below the documented minimum, so I used an existing Python 3.13.15 interpreter to create the venv. `pip check` returned `No broken requirements found.`

From a fresh directory, with Python 3.13.15 available as `python3`, run:

```bash
git clone https://github.com/LifeBringer/pathreview-ai301-fa26-s1.git
cd pathreview-ai301-fa26-s1
git checkout f89c06fc3ff292df2a04a39ac51319d32a76b779
python3 -m venv .venv
.venv/bin/python -m pip install --upgrade pip setuptools wheel
.venv/bin/pip install -e '.[dev]'
.venv/bin/python -m pip check
```

All commands below run from that repository root. The dependency manifest uses version ranges, so compare your resolved versions with those above if results differ. These probes need no `.env`, API key or application-specific environment variable. Docker was unavailable here, so I did not run the full web app, PostgreSQL/Redis setup, migrations or paid embedding services. The offline ingestion check below states its substituted boundaries explicitly.

### 1. Exact issue example

```bash
.venv/bin/python -c "from ingestion.chunking.structural_chunker import StructuralChunker; c = StructuralChunker(); print(len(c.chunk('This is a plain document with no headings at all. ' * 20, {})))"
```

Observed stdout and exit status:

```text
0
exit 0
```

Expected: the nonempty document should produce at least one chunk, retaining its content through a single block or a fallback. Actual: no chunks, so none of its content is available to the next stage. This is not an import/setup failure.

### 2. Repetition, headed control and real parser/strategy path

```bash
.venv/bin/python - <<'PY'
from ingestion.chunking.structural_chunker import StructuralChunker
from ingestion.chunking.strategy_selector import StrategySelector
from ingestion.parsers.readme_parser import ReadmeParser

text = 'This is a plain document with no headings at all. ' * 20
headed = '# Title\n' + text
chunker = StructuralChunker()
print('input_chars=', len(text), 'input_tokens=', len(chunker.encoder.encode(text)))
for attempt in range(1, 4):
    for name, value in [('plain', text), ('headed', headed)]:
        chunks = chunker.chunk(value, {})
        print(attempt, name, 'chunks=', len(chunks),
              'retains_body=', any(c.text == text.strip() for c in chunks))
print('plain_sections=', chunker._extract_sections(text))
print('headed_section_paths=', [s['path'] for s in chunker._extract_sections(headed)])
selector = StrategySelector()
for name, value in [('plain', text), ('headed', headed)]:
    parsed = ReadmeParser().parse(value)
    chunks = selector.chunk(parsed.text, parsed.metadata)
    print('readme_path', name, 'parser_preserves_input=', parsed.text == value,
          'heading_count=', parsed.metadata['heading_count'], 'chunks=', len(chunks))
chunks = selector.chunk(text, {'source_type': 'resume'})
print('same_plain_text_resume_strategy_chunks=', len(chunks))
PY
```

Observed stdout (exit 0):

```text
input_chars= 1000 input_tokens= 221
1 plain chunks= 0 retains_body= False
1 headed chunks= 1 retains_body= True
2 plain chunks= 0 retains_body= False
2 headed chunks= 1 retains_body= True
3 plain chunks= 0 retains_body= False
3 headed chunks= 1 retains_body= True
plain_sections= []
headed_section_paths= [['Title']]
readme_path plain parser_preserves_input= True heading_count= 0 chunks= 0
readme_path headed parser_preserves_input= True heading_count= 1 chunks= 1
same_plain_text_resume_strategy_chunks= 1
```

The initiating input is the nonempty body without a recognized Markdown heading. The relevant strategy condition is structural chunking, selected for `source_type='readme'`. The visible symptom is zero output chunks despite unchanged input surviving the README parser. Changing only the strategy to the resume/semantic path retains a chunk, so this does not establish loss for every document type.

### 3. Named test and the actual failing assertion

```bash
.venv/bin/python -m pytest tests/unit/test_structural_chunker.py -k test_document_with_no_headings -rxX -vv
.venv/bin/python -m pytest tests/unit/test_structural_chunker.py::TestStructuralChunker::test_document_with_no_headings --runxfail -q
.venv/bin/python -m pytest tests/unit/test_structural_chunker.py -rxX -q
```

The first command returned `14 deselected, 1 xfailed`, exit 0. The repository deliberately marks this seeded defect as a strict xfail. The second command temporarily ignores the marker without editing the test and returned exit 1:

```text
>       assert len(result) >= 1
E       assert 0 >= 1
E        +  where 0 = len([])

tests/unit/test_structural_chunker.py:36: AssertionError
1 failed in 0.12s
```

The complete structural-chunker test file returned `14 passed, 1 xfailed in 0.13s`, exit 0. I did not remove the marker, repair the intentional failure or change production code.

### 4. Offline ingestion-to-storage check

This exercises the original `IngestionPipeline.ingest_readme`, parser, selector and batch processor against in-memory Chroma. It uses the repository's deterministic `MockEmbeddingProvider`, not a paid model. The database session is a test double that reports no existing source. This is not a browser/API/PostgreSQL end-to-end test.

```bash
.venv/bin/python - <<'PY'
from unittest.mock import Mock
import chromadb
from chromadb.config import Settings
from ingestion.embeddings.provider import MockEmbeddingProvider
from ingestion.pipeline import IngestionPipeline

text = 'This is a plain document with no headings at all. ' * 20
client = chromadb.EphemeralClient(settings=Settings(anonymized_telemetry=False))
for name, value in [('plain', text), ('headed', '# Title\n' + text)]:
    collection = client.create_collection('issue56_' + name)
    session = Mock()
    session.query.return_value.filter_by.return_value.first.return_value = None
    pipeline = IngestionPipeline(collection, session, MockEmbeddingProvider())
    result = pipeline.ingest_readme('local-profile', name, value)
    print('RESULT', name, 'chunk_count=', result.chunk_count,
          'skipped=', result.skipped, 'stored_count=', collection.count(),
          'retains_body=', text.strip() in collection.get()['documents'])
PY
```

Result lines (exit 0):

```text
RESULT plain chunk_count= 0 skipped= False stored_count= 0 retains_body= False
RESULT headed chunk_count= 1 skipped= False stored_count= 1 retains_body= True
```

The heading-free run logged `README chunked successfully` with `chunk_count=0`, then `Empty chunks list provided to BatchEmbeddingProcessor`, then `README embeddings stored` with `chunk_count=0`. Thus the returned ingestion result does not signal a failure and the collection is empty, but there **is** a warning in the batch-processor log. I am not claiming complete silence in all logs.

### Explanation and limits

The earliest demonstrated loss is in section extraction, not README parsing or embedding. In [the tested structural chunker](https://github.com/LifeBringer/pathreview-ai301-fa26-s1/blob/f89c06fc3ff292df2a04a39ac51319d32a76b779/ingestion/chunking/structural_chunker.py#L118-L133), regular lines are collected only when `heading_stack or current_section_lines` is true. With no heading both remain empty; the final emission also requires a heading stack. The empty extracted sections and the headed control support this explanation. The input is only 221 tokens, below the 800-token sub-chunking threshold. Git blame traces the collection/emission guards to `10d3713b`; I did not execute a historical revision or infer causality from the newest comment about the seeded defect.

This confirms the supplied example and the named assertion at the stated revision, plus content loss through the bounded offline ingestion path. It does not establish behavior on other OS/interpreter versions or the deployed web app. My scope here is investigation and reporting, not a fix or delivery promise.

AI assistance: an assistant helped draft this report and execute and inspect the documented checks on my behalf. These are independent runs for this assignment, not another commenter's reproduction copied as mine.

---

## Eval iterations

**Run history**

I used the unchanged official Unit 2 harness and its pinned Sonnet model. Every run read the canonical installed `rubric.md`, `references/evidence-guide.md` and `SKILL.md` under `~/.claude/skills/repro-check/`. These were the actual evaluations on September 28, 2026:

1. Full v1: `agreement: 18/20 scored items  (bar: 18/20: below the bar; category floor unmet: no match in disclosure)`. Categories: `clear-accept 7/8  disclosure 0/1  no-evidence 4/4  unfollowable-comms 3/3  wrong-target 4/4`. I inspected both disagreements. `pkg-05` was falsely rejected for not repeating full `conda info`/`conda list` dumps in a follow-up with adequate trigger evidence. `pkg-20` was falsely accepted because the grader interpreted missing disclosure as no AI use. I changed only Repo conventions and its Comms evidence map: separate original-issue template prompts from explicit comment policies, and make the course's AI-assisted-candidate premise explicit.
2. Focused v2, `--only pkg-05,pkg-20,pkg-07,calib-03 --include-calibration`: `agreement: 3/3 scored items`. Both disagreements then matched. `pkg-20` was the essential sole-category disclosure canary affected by the Comms revision; it had not agreed on v1. Already-agreeing `pkg-07` checked that valid disclosure still passes. Unscored `calib-03` remained reject as a wrong-target trap. This partial run was not the submitted eval file and made no claim about the full category floor.
3. Confirming full v2, `--save-run eval-run.txt`: `agreement: 19/20 scored items  (bar: 18/20: PASS)`. Final categories: `clear-accept 7/8  disclosure 1/1  no-evidence 4/4  unfollowable-comms 3/3  wrong-target 4/4`. All category floors passed. The only disagreement was `pkg-09`, discussed below. I left every skill input unchanged after this run and copied the original harness-written output without editing it.

All three runs had zero item errors. There were no other full, smoke, limited or manual grading runs. I read `calib-02` during preparation, but did not grade it or count it as a run. No personal Unit 2 worksheet was supplied; the checks came from the official proof families and the actual evaluation feedback, not invented activity notes.

The first actual live claim check accepted the draft with two passes and four repro-only checks marked `not yet applicable: claim-only draft`. I posted that exact claim at `2026-09-28T05:49:26Z`, before the first reproduction at `05:55:14Z`. The first full-package live check also accepted, with all six checks passing. It encountered one denied optional compound shell command while checking source lines, then completed using permitted read-only evidence; no live rerun or skill change was needed. Both invoked the installed Skill and read its files. Raw outputs are retained privately. Its claim summary miscounted existing classmate comments and its full-package summary described the generic assistant disclosure as tool-and-extent detail; neither statement is needed for the actual no-disclosure-policy verdict, and neither is used as reproduction evidence here. The checked drafts and actual posted bodies matched byte-for-byte.

**Package analysis**

In the final scored run, `pkg-20` had rubric verdict `reject` and gold verdict `reject`:

```text
pkg-20  reject  reject   yes
```

The frozen policy says: "All AI usage in any form must be disclosed, stating the tool used and the extent of the assistance" and "AI-assisted issues and comments must be reviewed and edited by a human before submission". The candidate supplies strong technical proof: its single-theme query returns `^[[?997;2n`, while the conditional-theme control returns `^[[?997;1n`. But neither candidate comment discloses AI use. The gold note states that "course packages are treated as AI-assisted work". My revised policy check therefore rejects the missing disclosure without pretending the technical reproduction is weak. V1's opposite verdict was a real mistake: silence in the comment cannot waive a policy that applies to this course package.

**Check rationale**

This is the exact current Repo conventions row uploaded in `tools/repro-check/rubric.md`:

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| Repo conventions | Repo-facts contribution policy and report-template asks against candidate comments; live policy locations in Comms. | The comments satisfy applicable explicit contribution policies, including any AI-use ban or required disclosure in its stated location and form. Treat course eval candidates as AI-assisted work; silence in a candidate is not evidence of no AI use. If policy requires all AI use disclosed with tool and extent, those details must appear. Policy silence or permission-with-responsibility does not itself require disclosure. For follow-up comments on an existing issue, report-template asks are evidence prompts, not a demand to repeat the original report: accept equivalent trigger-relevant information without redundant diagnostic dumps or duplicate-search boilerplate unless policy explicitly requires them in every comment. In claim-only mode defer asks needing a repro report. | required |

I revised this row after the two v1 disagreements. The evidence sources are explicit: the frozen repo-facts policy/template asks in eval mode, or current official contribution and issue-template locations in live mode, compared with the actual comments. The threshold is compliance with applicable explicit policies, not a count of headings or pasted diagnostics. That lets a sufficient follow-up avoid repeating irrelevant original-issue boilerplate while retaining a strict disclosure requirement when a repository actually has one. I rejected both blanket mandatory AI disclosure on policy-silent repositories and the assumption that an undisclosed course candidate must be unassisted.

**Trade-offs**

My final run still falsely rejects `pkg-09`, whose gold label is `accept`. It gives a concrete failed attempt at `fd --exec-batch` argument-size reordering, shows the marker sequence `ONE ONE ONE TWO TWO TWO`, and explains that its padding may not have made one command reach the size limit first. The grader marked Right behavior proved `unclear` because the asymmetric trigger might not have been reached. I understand the gold acceptance: an honest, evidenced cannot-reproduce report can explain that very limitation without claiming the bug is absent.

That check's wording was unchanged between the first and final full runs, yet this package flipped from accept to reject. This exposes a real risk of an overly strict or variable reading of "actual trigger" when attempted reproduction cannot establish every hidden condition. I retained the passing, non-perfect run rather than claiming 20/20 or tuning to force a result. Conversely, loosening trigger fidelity indiscriminately would let adjacent failures count as reproduction. My own report makes the tested boundary explicit: real chunker/parser/ingestion code, in-memory Chroma, mock embeddings and a mocked metadata database, not the deployed web app. Its batch warning also limits the issue title's word "silently"; I report that contrary detail instead of omitting it.
