# Unit 3 - Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

---

## Posted upstream

**GitHub username**

LifeBringer

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/56#issuecomment-5987275154

## Plan for #56: keep heading-free documents as one section

This plan builds on [my reproduction above](https://github.com/codepath/pathreview-ai301-fa26-s1/issues/56#issuecomment-5864368691). On 2026-10-05 I re-ran the same steps at `f89c06f`, still upstream `main`, in a fresh Python 3.13.15 venv and got the same results:

```text
exact issue example: 0
1 plain chunks= 0 retains_body= False
1 headed chunks= 1 retains_body= True
plain_sections= []
readme_path plain parser_preserves_input= True heading_count= 0 chunks= 0
same_plain_text_resume_strategy_chunks= 1
RESULT plain chunk_count= 0 skipped= False stored_count= 0 retains_body= False
RESULT headed chunk_count= 1 skipped= False stored_count= 1 retains_body= True
```

**Diagnosis.** In `StructuralChunker._extract_sections()` (`ingestion/chunking/structural_chunker.py`), a content line is collected only when `heading_stack or current_section_lines` is true (line 120), and the final section is saved only when `heading_stack` is set (line 124). Without a heading, nothing is collected and the method returns `[]`. This fits the evidence: the README parser keeps the text (`heading_count= 0`), the semantic strategy keeps it, only a leading `# Title` changes the result, and at 221 tokens the 800-token sub-chunking path is not involved. The empty Chroma collection follows from receiving zero chunks.

**Change.** Two files: `ingestion/chunking/structural_chunker.py` and `tests/unit/test_structural_chunker.py`.

1. In `_extract_sections()`, collect every content line and save the final section whenever it has content. A heading-free document becomes one section with an empty path and level 0, which the existing section dict already produces when the heading stack is empty. `chunk()` then emits one chunk, or sub-chunks through the existing semantic path above 800 tokens.
2. Remove the unused `current_level` variable with its `Seeded defect (issue #56) ... Leave as-is` comment and `# noqa: F841`. That annotation marks this issue's own defect, and after the fix its statement that heading-less documents get dropped is no longer true. The annotations for other seeded defects stay untouched.
3. Following the seeded-bug rule in `docs/CONTRIBUTING.md`, delete the `xfail` marker from `test_document_with_no_headings`, and add two tests: the issue's input gives one chunk equal to the stripped input with `heading_path ""` and `heading_level 0`, and heading-free text over 800 tokens gives several chunks that keep its first and last sentences.

**Not changing.** Any document with at least one heading keeps exactly its current sections. That includes text before the first heading, which a pre-build probe shows is also dropped today (`preamble_then_heading ... chunks= 1 ... has_intro= False`). Issue #56 does not report that case, so I would raise it separately. The heading pattern, `heading_path` format, token limit, `SemanticChunker`, selector, parser and ingestion pipeline also stay as they are.

**Test plan.** Re-run the reproduction against the change and expect: the exact example prints `1`; each comparison shows `plain chunks= 1 retains_body= True` with the headed line unchanged; `readme_path plain ... chunks= 1`; `test_document_with_no_headings` passes on its real assertions with no marker; the structural test file gives 17 passed, 0 xfailed; the new tests fail on the unfixed code and pass after; and the offline ingestion check gives `RESULT plain chunk_count= 1 skipped= False stored_count= 1 retains_body= True`. Also `ruff check .`, `black --check .`, mypy and `pytest tests/unit -m unit`, which gave `375 passed, 53 xfailed` at the base and should give `378 passed, 52 xfailed`.

**Risks and unknowns.** I have not checked how an empty `heading_path` shows up in retrieval output; the ingestion re-run checks that Chroma stores it. Local runs use Python 3.13.15 while CI uses 3.11, and without Docker there is no full app or database run. The ingestion check keeps its mock embeddings, mocked metadata session and in-memory Chroma.

I plan to build this on `fix/56-keep-heading-free-docs` in my fork. I am not promising a date.

AI assistance: an assistant helped draft this plan and will help implement and test the change on my behalf. The plan comes from my own reproduction, not another commenter's.

---

## Your branch

**Branch**

fix/56-keep-heading-free-docs

https://github.com/LifeBringer/pathreview-ai301-fa26-s1/tree/fix/56-keep-heading-free-docs, commit `db5b3a14d00392d35720988fdc3ca705db9c2258`, one commit on top of `f89c06fc3ff292df2a04a39ac51319d32a76b779`.

**Evidence**

All commands ran from the root of my fork clone on macOS 27.0 (arm64) with Python 3.13.15 in a fresh venv set up with `pip install -e '.[dev]'`, as in my posted reproduction. Before is the unchanged base `f89c06fc3ff292df2a04a39ac51319d32a76b779`, still upstream `main`, re-run at 2026-10-05T02:28:30Z; it reproduces every result in the posted Unit 2 output, with only timings and log timestamps differing. After is `db5b3a14d00392d35720988fdc3ca705db9c2258` on the branch above, run at 2026-10-05T02:55:03Z. `repro_probe.py` and `repro_ingestion.py` are the step 2 and step 4 scripts from the posted reproduction, saved unchanged as files outside the repository. Pytest runs use `-p no:cacheprovider` so no cache lands in the work tree. In the `-vv` output I left out pytest's session header (platform, plugin and local path lines); everything else is pasted as printed.

1. Exact issue example

```bash
.venv/bin/python -c "from ingestion.chunking.structural_chunker import StructuralChunker; c = StructuralChunker(); print(len(c.chunk('This is a plain document with no headings at all. ' * 20, {})))"
```

Before (exit 0):

```text
0
```

After (exit 0):

```text
1
```

2. Repeated plain versus headed comparison, README parser and strategy path

```bash
.venv/bin/python ../repro_probe.py
```

Before (exit 0):

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

After (exit 0):

```text
input_chars= 1000 input_tokens= 221
1 plain chunks= 1 retains_body= True
1 headed chunks= 1 retains_body= True
2 plain chunks= 1 retains_body= True
2 headed chunks= 1 retains_body= True
3 plain chunks= 1 retains_body= True
3 headed chunks= 1 retains_body= True
plain_sections= [{'content': 'This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all. This is a plain document with no headings at all.', 'path': [], 'level': 0}]
headed_section_paths= [['Title']]
readme_path plain parser_preserves_input= True heading_count= 0 chunks= 1
readme_path headed parser_preserves_input= True heading_count= 1 chunks= 1
same_plain_text_resume_strategy_chunks= 1
```

3. The named test, run three ways

```bash
.venv/bin/python -m pytest tests/unit/test_structural_chunker.py -k test_document_with_no_headings -rxX -vv -p no:cacheprovider
.venv/bin/python -m pytest tests/unit/test_structural_chunker.py::TestStructuralChunker::test_document_with_no_headings --runxfail -q -p no:cacheprovider
.venv/bin/python -m pytest tests/unit/test_structural_chunker.py -rxX -q -p no:cacheprovider
```

Before, with the strict xfail marker still on the test (exits 0, 1, 0):

```text
collecting ... collected 15 items / 14 deselected / 1 selected

tests/unit/test_structural_chunker.py::TestStructuralChunker::test_document_with_no_headings XFAIL [100%]

=========================== short test summary info ============================
XFAIL tests/unit/test_structural_chunker.py::TestStructuralChunker::test_document_with_no_headings - issue #56: structural chunker drops documents with no headings
====================== 14 deselected, 1 xfailed in 0.75s =======================
```

```text
F                                                                        [100%]
=================================== FAILURES ===================================
_____________ TestStructuralChunker.test_document_with_no_headings _____________

self = <tests.unit.test_structural_chunker.TestStructuralChunker object at 0x107286c40>
chunker = <ingestion.chunking.structural_chunker.StructuralChunker object at 0x107375d30>

    @pytest.mark.xfail(
        strict=True, reason="issue #56: structural chunker drops documents with no headings"
    )
    def test_document_with_no_headings(self, chunker):
        """Test document with no headings returns single chunk."""
        text = "This is plain text without any markdown headings. " * 20
        result = chunker.chunk(text, {"source": "test"})
    
>       assert len(result) >= 1
E       assert 0 >= 1
E        +  where 0 = len([])

tests/unit/test_structural_chunker.py:36: AssertionError
=========================== short test summary info ============================
FAILED tests/unit/test_structural_chunker.py::TestStructuralChunker::test_document_with_no_headings
1 failed in 0.15s
```

```text
..x............                                                          [100%]
=========================== short test summary info ============================
XFAIL tests/unit/test_structural_chunker.py::TestStructuralChunker::test_document_with_no_headings - issue #56: structural chunker drops documents with no headings
14 passed, 1 xfailed in 0.16s
```

After, with the marker removed by the change, so the test runs its real assertions (exits 0, 0, 0). The `-k` filter now also selects the new `test_document_with_no_headings_keeps_text`:

```text
collecting ... collected 17 items / 15 deselected / 2 selected

tests/unit/test_structural_chunker.py::TestStructuralChunker::test_document_with_no_headings PASSED [ 50%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_document_with_no_headings_keeps_text PASSED [100%]

======================= 2 passed, 15 deselected in 0.14s =======================
```

```text
.                                                                        [100%]
1 passed in 0.14s
```

```text
.................                                                        [100%]
17 passed in 0.15s
```

Tests first: the changed test file against the unfixed production code (base source, only the test file edited), before the source change. The three heading-free tests fail on their real assertions (exit 1); assertion lines shown:

```bash
.venv/bin/python -m pytest tests/unit/test_structural_chunker.py -rxX -q -p no:cacheprovider
```

```text
..FFF............                                                        [100%]
_____________ TestStructuralChunker.test_document_with_no_headings _____________
>       assert len(result) >= 1
E       assert 0 >= 1
E        +  where 0 = len([])
tests/unit/test_structural_chunker.py:33: AssertionError
_______ TestStructuralChunker.test_document_with_no_headings_keeps_text ________
>       assert len(result) == 1
E       assert 0 == 1
E        +  where 0 = len([])
tests/unit/test_structural_chunker.py:42: AssertionError
__ TestStructuralChunker.test_large_document_with_no_headings_is_sub_chunked ___
>       assert len(result) > 1
E       assert 0 > 1
E        +  where 0 = len([])
tests/unit/test_structural_chunker.py:56: AssertionError
3 failed, 14 passed in 0.33s
```

4. Offline ingestion-to-storage check. It runs the original `IngestionPipeline.ingest_readme`, parser, selector and batch processor with the repository's `MockEmbeddingProvider`, a mocked metadata session and in-memory Chroma. It is not a browser, API or PostgreSQL run.

```bash
.venv/bin/python ../repro_ingestion.py
```

Before (exit 0):

```text
2026-10-04 22:28:42 [info     ] Starting README ingestion      profile_id=local-profile repo_name=plain source_id=readme_local-profile_plain_39a1da02872c4976
2026-10-04 22:28:42 [info     ] README parsed successfully     heading_count=0 word_count=200
2026-10-04 22:28:42 [info     ] README chunked successfully    chunk_count=0
2026-10-04 22:28:42 [warning  ] Empty chunks list provided to BatchEmbeddingProcessor
2026-10-04 22:28:42 [info     ] README embeddings stored       chunk_count=0
2026-10-04 22:28:42 [info     ] Recording ingested source      chunk_count=0 profile_id=local-profile source_id=readme_local-profile_plain_39a1da02872c4976 source_type=readme
RESULT plain chunk_count= 0 skipped= False stored_count= 0 retains_body= False
2026-10-04 22:28:42 [info     ] Starting README ingestion      profile_id=local-profile repo_name=headed source_id=readme_local-profile_headed_4b0df80979ca3310
2026-10-04 22:28:42 [info     ] README parsed successfully     heading_count=1 word_count=202
2026-10-04 22:28:42 [info     ] README chunked successfully    chunk_count=1
2026-10-04 22:28:42 [info     ] Starting batch embedding processing chunk_count=1
2026-10-04 22:28:42 [info     ] Processing embedding batch     batch_end=1 batch_num=1 batch_start=0 total=1
2026-10-04 22:28:44 [info     ] Generated embeddings for batch embedding_count=1
2026-10-04 22:28:44 [debug    ] Stored embedding in vector DB  embedding_id=readme_local-profile_headed_4b0df80979ca3310_chunk_0
2026-10-04 22:28:44 [info     ] Batch embedding processing complete stored_count=1
2026-10-04 22:28:44 [info     ] README embeddings stored       chunk_count=1
2026-10-04 22:28:44 [info     ] Recording ingested source      chunk_count=1 profile_id=local-profile source_id=readme_local-profile_headed_4b0df80979ca3310 source_type=readme
RESULT headed chunk_count= 1 skipped= False stored_count= 1 retains_body= True
```

After (exit 0). The plain document is stored and the empty-chunks warning is gone:

```text
2026-10-04 22:55:05 [info     ] Starting README ingestion      profile_id=local-profile repo_name=plain source_id=readme_local-profile_plain_39a1da02872c4976
2026-10-04 22:55:05 [info     ] README parsed successfully     heading_count=0 word_count=200
2026-10-04 22:55:05 [info     ] README chunked successfully    chunk_count=1
2026-10-04 22:55:05 [info     ] Starting batch embedding processing chunk_count=1
2026-10-04 22:55:05 [info     ] Processing embedding batch     batch_end=1 batch_num=1 batch_start=0 total=1
2026-10-04 22:55:05 [info     ] Generated embeddings for batch embedding_count=1
2026-10-04 22:55:05 [debug    ] Stored embedding in vector DB  embedding_id=readme_local-profile_plain_39a1da02872c4976_chunk_0
2026-10-04 22:55:05 [info     ] Batch embedding processing complete stored_count=1
2026-10-04 22:55:05 [info     ] README embeddings stored       chunk_count=1
2026-10-04 22:55:05 [info     ] Recording ingested source      chunk_count=1 profile_id=local-profile source_id=readme_local-profile_plain_39a1da02872c4976 source_type=readme
RESULT plain chunk_count= 1 skipped= False stored_count= 1 retains_body= True
2026-10-04 22:55:05 [info     ] Starting README ingestion      profile_id=local-profile repo_name=headed source_id=readme_local-profile_headed_4b0df80979ca3310
2026-10-04 22:55:05 [info     ] README parsed successfully     heading_count=1 word_count=202
2026-10-04 22:55:05 [info     ] README chunked successfully    chunk_count=1
2026-10-04 22:55:05 [info     ] Starting batch embedding processing chunk_count=1
2026-10-04 22:55:05 [info     ] Processing embedding batch     batch_end=1 batch_num=1 batch_start=0 total=1
2026-10-04 22:55:05 [info     ] Generated embeddings for batch embedding_count=1
2026-10-04 22:55:05 [debug    ] Stored embedding in vector DB  embedding_id=readme_local-profile_headed_4b0df80979ca3310_chunk_0
2026-10-04 22:55:05 [info     ] Batch embedding processing complete stored_count=1
2026-10-04 22:55:05 [info     ] README embeddings stored       chunk_count=1
2026-10-04 22:55:05 [info     ] Recording ingested source      chunk_count=1 profile_id=local-profile source_id=readme_local-profile_headed_4b0df80979ca3310 source_type=readme
RESULT headed chunk_count= 1 skipped= False stored_count= 1 retains_body= True
```

5. Repository checks

```bash
.venv/bin/ruff check .
.venv/bin/black --check .
.venv/bin/mypy api/ core/ ingestion/ rag/ agent/ safety/
.venv/bin/pytest tests/unit -q -m unit -p no:cacheprovider    # before
.venv/bin/pytest tests/unit -v -m unit -p no:cacheprovider    # after
.venv/bin/pytest tests/integration -v -p no:cacheprovider     # after
```

Before:

```text
All checks passed!
110 files would be left unchanged.
Success: no issues found in 76 source files
375 passed, 53 xfailed, 1 warning in 11.08s
```

After:

```text
All checks passed!
110 files would be left unchanged.
Success: no issues found in 76 source files
================== 378 passed, 52 xfailed, 1 warning in 6.66s ==================
============================ no tests ran in 0.08s =============================
```

The integration run exits 5 because `tests/integration` holds no tests at this revision; the repository's CI treats that as success. The one warning is the existing Pydantic class-based config deprecation in `core/config.py`.

6. Edge inputs from the plan (not part of the posted reproduction), each through `StructuralChunker().chunk(value, {})`, plus a comparison of base and fixed output on every tracked Markdown file with a heading:

```bash
.venv/bin/python ../scope_probe.py
.venv/bin/python ../headed_invariance.py ../base_structural_chunker.py
```

Before:

```text
large_plain tokens= 1101 chunks= 0 paths= [] has_intro= False
hash_without_space tokens= 225 chunks= 0 paths= [] has_intro= False
preamble_then_heading tokens= 230 chunks= 1 paths= ['Title'] has_intro= False
whitespace_only tokens= 2 chunks= 0 paths= [] has_intro= False
```

After:

```text
large_plain tokens= 1101 chunks= 3 paths= [''] has_intro= False
hash_without_space tokens= 225 chunks= 1 paths= [''] has_intro= False
preamble_then_heading tokens= 230 chunks= 1 paths= ['Title'] has_intro= False
whitespace_only tokens= 2 chunks= 0 paths= [] has_intro= False
headed_markdown_files_compared= 16 identical_chunks= 16
```

Not run: Docker, the full web app, PostgreSQL, Redis, the frontend tests (the frontend is untouched) and Python 3.11, the CI version. CI first runs on the Unit 4 pull request.

## Eval iterations

**Run history**

Every run used the unchanged official harness from `codepath/ai301-unit3-starter` at `38a9c48e49c39f84b281ed523f74847ec563daaf`, its pinned Sonnet grader and the installed files in `~/.claude/skills/plan-check/`. In order, on 2026-10-05:

1. Full run, v1: `agreement: 19/20 scored items  (bar: 18/20: PASS)`. `categories: clear-accept 6/7  scope-creep 4/4  thread-convention 2/2  unbuildable 3/3  wrong-cause 4/4`. Zero errors. The one disagreement was pkg-09, a clear accept rejected on Thread and conventions because the plan credits a non-maintainer's "simpler route" remark to the collaborator. I revised only that check and the matching evidence-guide sentences: it now grades what the comment does about direction and policy, and attribution precision moved to the preferred Calibrated claims.
2. Targeted retry, `--only pkg-09,pkg-20,pkg-04`: `agreement: 3/3 scored items`. pkg-09 then matched. pkg-04 and pkg-20, the only two thread-convention packages and both already agreeing, stayed reject as canaries for the loosened check. This partial run did not touch `eval-run.txt`.
3. Full run, v2, with `--save-run eval-run.txt`: `agreement: 20/20 scored items  (bar: 18/20: PASS)`. `categories: clear-accept 7/7  scope-creep 4/4  thread-convention 2/2  unbuildable 3/3  wrong-cause 4/4`. Zero errors. This is the committed `eval-run.txt`, and no skill file changed after it.

The live plan-check runs on my own drafts were not eval runs: the installed skill accepted the drafts before posting (all six checks pass) and accepted the as-built plan with Deviations filled against the posted comment.

**Package analysis**

pkg-09 (`sharkdp/fd#2067`, category clear-accept). In the final run my rubric decided `accept`, and the gold label is `accept`:

```text
pkg-09  clear-accept       accept  accept   yes
```

The repro shows `fd --glob --full-path "**/src/**/*.spec.ts" C:\t\fixture` "prints nothing, exit 0" on native Windows, while the regex-mode control prints `C:\t\fixture\src\foo\a.spec.ts` and the Linux control matches. The plan's cause, a glob-derived regex that "expects `/` separators but is matched against raw native paths with `\`", explains both controls, so Grounded diagnosis passes. The change is one Windows-gated normalization where "walk.rs matches the pattern regex against the full path", with option 1 among its "stated deferrals", so Bounded scope and Executable approach pass. The test plan re-runs "the three pattern spellings from the repro" and expects the fixture path, a result that differs from today's empty output. For the thread, tmccombs (COLLABORATOR) listed three options without choosing one; the comment picks option 2, names PR #2089 and offers to rebase onto it. The repo facts say "the policy states no disclosure ask for issue comments", so Thread and conventions passes.

My v1 rubric rejected this package. The plan says option 2 is what "the collaborator noted is the simpler route", but the thread shows "the simpler and less regex-fragile route" came from petrroll (NONE); the collaborator only wrote that option 2 "could simplify other code paths". That is a loose attribution in the plan, not ignored direction in the comment, so v2 grades it under Calibrated claims, which never changes the verdict. I agree with the gold label: a stranger could build this plan and its comment respects the thread.

**Check rationale**

The exact current Thread and conventions row in `tools/plan-check/rubric.md`:

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| Thread and conventions | The plan comment, with the plan, read against explicit maintainer statements in the thread and the repo's stated contribution and AI-use policy (Comms). | (a) Where an owner, member, collaborator or maintainer has stated direction relevant to this work (an approach wanted or ruled out, where the fix belongs, whether a change is wanted, who is already doing it), the comment follows it or names it and explains the difference so the maintainer can decide. Claims from non-maintainer commenters are not direction. (b) Where the stated policy requires something of comments or of the planned work (AI-use disclosure, human-written comments, a required step before a change), the comment meets it in the stated form. Treat course packages as AI-assisted work: silence about AI use does not meet a disclosure requirement. With no maintainer direction and no applicable policy ask, this check passes. Bug-report template fields are asks for the original report, not for a plan comment. This check grades what the comment does about direction and policy, not how precisely the plan paraphrases or attributes thread remarks; a loose or misattributed summary that does not change whether the direction is followed belongs to Calibrated claims. | required |

It reads as two outcome tests because the eval's thread-convention category describes two different failures: a comment that ignores explicit maintainer direction, and one that omits disclosure the repo's AI policy requires. Both clauses are tied to the role in the thread highlights and the wording in the repo facts, so another grader can find the same evidence. I rejected treating every commenter's claim as direction, because classmates and passers-by also post approaches; only owner, member, collaborator or maintainer statements bind. The AI premise and the template sentence come from my Unit 2 lessons: there, silence was misread as no AI use, and a follow-up comment was failed for not repeating the original bug template. The last sentence is the v2 revision. In v1, pkg-09 failed this check over who first called option 2 simpler, which is accuracy of a summary, not a choice that ignores a maintainer. I kept the required check focused on what the comment does and moved attribution accuracy to the preferred Calibrated claims rather than dropping it.

**Trade-offs**

The v2 sentence gives up strictness on misattribution. A comment that follows the real direction but credits its approach to the wrong person now passes this required check, and only the preferred Calibrated claims reports it. pkg-09 is exactly that case, and I accept that this rubric will pass similar plans. To check that the loosening did not let real violations through, I re-ran `--only pkg-09,pkg-20,pkg-04`. pkg-20 (a required AI disclosure missing from the comment) and pkg-04 (the owner's ongoing code fix not acknowledged) both stayed reject, because their failures are about what the comment does, not attribution. The confirming full run then kept every package that agreed in v1 in agreement and reached 20/20.

---

Related paths: `plan.md` and `eval-run.txt` in this directory; the skill's files in `tools/plan-check/`.
