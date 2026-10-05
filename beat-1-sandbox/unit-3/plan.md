# Plan: keep heading-free documents in StructuralChunker (issue #56)

- Issue: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/56
- Reproduction this plan builds on: my posted report, https://github.com/codepath/pathreview-ai301-fa26-s1/issues/56#issuecomment-5864368691
- Base: `f89c06fc3ff292df2a04a39ac51319d32a76b779`, the tested revision, which is still upstream `main` and my fork's `main`.
- Planned branch: `fix/56-keep-heading-free-docs` in https://github.com/LifeBringer/pathreview-ai301-fa26-s1

## Evidence relied on

From the posted reproduction (Python 3.13.15, macOS 27.0 arm64, `pip install -e '.[dev]'`):

- Step 1, the issue's exact example (`'This is a plain document with no headings at all. ' * 20`, 1000 characters, 221 tokens) printed `0`, exit 0.
- Step 2, three repeated comparisons where only a leading `# Title\n` differs:
  - `1 plain chunks= 0 retains_body= False`
  - `1 headed chunks= 1 retains_body= True`
  - `plain_sections= []` and `headed_section_paths= [['Title']]`
  - `readme_path plain parser_preserves_input= True heading_count= 0 chunks= 0`
  - `same_plain_text_resume_strategy_chunks= 1`
- Step 3, `test_document_with_no_headings` is a strict xfail; with `--runxfail` it fails on its real assertion: `assert 0 >= 1` / `where 0 = len([])`.
- Step 4, the original `IngestionPipeline.ingest_readme` with the repository's `MockEmbeddingProvider`, a mocked metadata session and in-memory Chroma: `RESULT plain chunk_count= 0 skipped= False stored_count= 0 retains_body= False` versus `RESULT headed chunk_count= 1 skipped= False stored_count= 1 retains_body= True`. The plain run logged `Empty chunks list provided to BatchEmbeddingProcessor`, a warning, while the returned result reports no failure.

I re-ran steps 1 to 4 unchanged on 2026-10-05 at the same revision in a fresh venv (Python 3.13.15, and the same versions of the dependencies listed in the report) and got the same lines.

Additional pre-build observations, same revision, not part of the posted reproduction. Each input goes through the real `StructuralChunker().chunk(value, {})`:

```text
large_plain tokens= 1101 chunks= 0 paths= [] has_intro= False
hash_without_space tokens= 225 chunks= 0 paths= [] has_intro= False
preamble_then_heading tokens= 230 chunks= 1 paths= ['Title'] has_intro= False
whitespace_only tokens= 2 chunks= 0 paths= [] has_intro= False
```

`large_plain` is the issue text repeated five times (over the 800-token sub-chunk limit). `hash_without_space` prefixes `#notaheading`. `preamble_then_heading` is `Intro line before any heading.\n\n# Title\n` plus the issue text.

## Diagnosis

- Initiating trigger: nonempty text with no line matching the heading pattern `^(#{1,6})\s+(.+)$`, chunked by `StructuralChunker` (the strategy `StrategySelector` picks for `source_type='readme'`).
- Masking condition: any recognized heading. Adding only `# Title\n` restores the chunk.
- Visible symptom: zero chunks, so ingestion stores nothing for that README and reports `skipped= False`.

Cause: in `StructuralChunker._extract_sections()` (`ingestion/chunking/structural_chunker.py`), a content line is appended only when `heading_stack or current_section_lines` is true (line 120). Without a heading both stay empty, so no line is ever collected. The final save also requires `heading_stack` (line 124). The method returns `[]`, and `chunk()` has no section to emit.

Why this fits every observation: `plain_sections= []` while the headed control yields `[['Title']]`; the README parser returns the input unchanged with `heading_count= 0`, so the loss is after parsing; the semantic strategy keeps the same text as one chunk, so the loss is specific to the structural path; the input is 221 tokens, under `SECTION_TOKEN_LIMIT = 800`, so sub-chunking is not involved. The empty Chroma collection in step 4 follows from receiving zero chunks; it is not a second cause. The pre-build `large_plain` result shows larger heading-free documents are lost the same way.

## Scope

In scope, one change: a document with no recognized heading becomes one section with an empty path and level 0, so `chunk()` emits it as one chunk, or sub-chunks it through the existing semantic path when it exceeds 800 tokens.

1. `_extract_sections()`: append every content line, and save the final section whenever it has content. The existing section dict already yields `path []` and `level 0` for an empty heading stack.
2. Remove `current_level` and its annotation (`# Seeded defect (issue #56) ... Leave as-is` and `# noqa: F841`). Nothing reads the variable before or after the fix, and the comment would become false. `docs/CONTRIBUTING.md` asks a seeded-bug fix to remove its marker and matching suppressions; this annotation names issue #56 itself, so it is not one of the other seeded defects the guide says to leave alone.
3. One docstring sentence in `_extract_sections()` stating the heading-free result.
4. Tests: delete the `@pytest.mark.xfail(strict=True, reason="issue #56: ...")` marker from `test_document_with_no_headings`, as `docs/CONTRIBUTING.md` requires, and add two regression tests (below).

Not changing:

- Text before the first heading of a document that has headings. The pre-build `preamble_then_heading` line shows that text is also dropped today. The mid-loop save keeps its `if heading_stack:` guard, so any document with at least one heading produces exactly the sections it produces now. That is a separate behavior that issue #56 does not report; I would raise it separately instead of folding it in.
- The heading pattern (`#notaheading` stays content), the `heading_path` format, `SECTION_TOKEN_LIMIT`, `SemanticChunker`, `StrategySelector`, the README parser, the ingestion pipeline, the empty or whitespace early return in `chunk()`, and every other seeded defect and annotation.

## Files

- `ingestion/chunking/structural_chunker.py`
- `tests/unit/test_structural_chunker.py`

## Approach and order

1. Create `fix/56-keep-heading-free-docs` from `f89c06f` in my fork clone.
2. Tests first. Remove the xfail marker. Add `test_document_with_no_headings_keeps_text`: the issue's exact input gives exactly one chunk whose text equals the stripped input, with `heading_path == ""`, `heading_level == 0` and the caller's metadata kept. Add `test_large_document_with_no_headings_is_sub_chunked`: heading-free text over 800 tokens gives more than one chunk, all with `heading_path == ""`, and its first and last sentences appear in the chunks. Run them against the unfixed code and keep the failing output.
3. Change `_extract_sections()` as in Scope items 1 to 3.
4. Run the test plan, then `ruff check .`, `black --check .` and `mypy api/ core/ ingestion/ rag/ agent/ safety/`.

## Test plan

Re-run the posted reproduction against the built change, same commands:

1. Exact issue example: expect `1` (was `0`), exit 0.
2. Step 2 script: every attempt `plain chunks= 1 retains_body= True` (was `0` / `False`); headed lines unchanged at `1` / `True`; `plain_sections` one section with `'path': []` and `'level': 0` (was `[]`); `readme_path plain ... chunks= 1` (was `0`); the resume strategy line stays `1`.
3. Named test without its marker: `pytest tests/unit/test_structural_chunker.py -k test_document_with_no_headings -rxX -vv` reports PASSED on the real assertions, not XFAIL or XPASS. The whole file: 17 passed, 0 xfailed (was 14 passed, 1 xfailed). The two new tests fail on the unfixed source and pass after.
4. Step 4 ingestion script: `RESULT plain chunk_count= 1 skipped= False stored_count= 1 retains_body= True` (was `0` / `0` / `False`), headed line unchanged, and no empty-chunks warning for the plain run. The boundary stays the same: mock embeddings, a mocked metadata session and in-memory Chroma, not the browser, API or PostgreSQL path.
5. Pre-build probe: `large_plain` gives 2 or more chunks with path `''`; `hash_without_space` gives 1; `preamble_then_heading` is unchanged (1 chunk, `['Title']`, `has_intro= False`); `whitespace_only` stays 0.
6. Repo checks: at the base, `ruff check .`, `black --check .` and mypy were clean and `pytest tests/unit -m unit` gave `375 passed, 53 xfailed`. Expect the same clean checks and `378 passed, 52 xfailed`.

## Risks and unknowns

- Heading-free chunks carry `heading_path ""` and `heading_level 0`. No code outside the chunker and its tests reads those keys, but I have not checked how a retrieval UI would display an empty path. Test plan step 4 checks that Chroma stores the chunk with that metadata.
- Large heading-free documents use `SemanticChunker` as large headed sections already do. Its overlap and offsets are unchanged; the new test checks only that the text arrives, not chunk boundaries.
- Local runs use Python 3.13.15; CI uses 3.11, which I do not have locally. Docker is unavailable, so there is no full app, PostgreSQL, Redis or browser run. The frontend is untouched and its tests will not be run. CI will first run on the Unit 4 pull request.
- The approach is mine, from my reproduction. Other students have posted plans on this issue; I have not copied theirs.

## Deviations

Built as commit `db5b3a14d00392d35720988fdc3ca705db9c2258` on `fix/56-keep-heading-free-docs`, from `f89c06f`. The production change, the removed `current_level` annotation, the docstring sentence, the removed xfail marker and the two named tests are as planned. What differs:

1. `test_large_document_with_no_headings_is_sub_chunked` checks that every one of its 150 generated sentences appears in the chunks, not only the first and last sentences as planned. It is a stronger check of the same claim.
2. I added two checks the test plan did not list. First, base and fixed `chunk()` output compared on all 16 tracked Markdown files that contain a heading: all 16 identical, which tests the claim that documents with a heading keep their sections. Second, `pytest tests/integration`, which collects no tests at this revision (exit 5, which the repository's CI treats as success).
3. Test plan step 3's `-k test_document_with_no_headings` filter now also selects the new `test_document_with_no_headings_keeps_text`, so it reports 2 passed instead of 1.

Every expected result in the test plan was observed, including the new tests failing first on the unfixed source. Nothing in the posted plan comment became untrue, so no follow-up comment is needed.
