# Unit 1 - Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/56

**Verdict output**

The calibrated skill ranked this first among its accepted candidates. I then ran it on this issue alone to confirm the selection. Below is that run's verbatim repository line, complete check table, summary, and final JSON. Only the display heading is omitted.

**Repo:** codepath/pathreview-ai301-fa26-s1 (in scope, live evidence gathered via gh-axi GET calls only)

| Check | Grade | Evidence |
|---|---|---|
| Open and writable | pass | `state: open`, `archived: false` (repo metadata, checked live) |
| Human maintenance | pass | Default-branch commit `f89c06f` authored by Andrew Burke (andrew@codepath.org), a human, dated 2026-09-16, 12 days before today (2026-09-28), well within 90 days |
| Recent use signal | pass | Repo `pushed_at: 2026-09-16T21:48:27Z`, 12 days ago, within the 180-day push threshold (no release needed) |
| Bounded contribution | pass | Single identifiable outcome: make `StructuralChunker.chunk()` handle heading-less documents (single block or fallback), with a named failing test `test_document_with_no_headings` already encoding the expected fix; no design debate, no maintainer requirement of a core parser/architecture rewrite; issue is 18 days old with no abandoned attempts |
| Available work | pass | `assignees: []`; no open implementation PR found (PR search for repo + "#56" returned `total_count: 0`); two "referenced" timeline events point to commits in an unrelated repo (`vchlinh/ai301-coursework`), not PRs against this issue; three student claim comments exist (himavanthkar, yitingzhang1113, LilRed92) but per the Path Review house rule in scope.md, classmates' claim comments do not block availability |
| AI-assisted workflow allowed | pass | `docs/CONTRIBUTING.md` (6662 bytes, fetched in full) contains no AI ban; no `AI_POLICY.md`/`AI_USAGE_POLICY.md`/`AGENTS.md` exists at repo root or in `.github/`; `.github/PULL_REQUEST_TEMPLATE.md` has no AI-disclosure requirement; silence passes per rubric |
| Verification anchor (preferred) | pass | Issue body contains an executable repro snippet (`StructuralChunker().chunk(...)` returning `0`) and names the exact regression test target `tests/unit/test_structural_chunker.py::test_document_with_no_headings` |

All six required checks pass, so the verdict is **accept**. The preferred verification-anchor check also passes, giving a concrete repro and named test target to build on, and this is a bounded Python bug fix in a single file, a good fit for Roy's stated preference for a bounded behavior fix with a concrete way to verify it.

```json
{
  "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/56",
  "checks": [
    {"name": "Open and writable", "grade": "pass", "evidence": "Issue state is open and repo archived flag is false as of live API fetch on 2026-09-28"},
    {"name": "Human maintenance", "grade": "pass", "evidence": "Default-branch commit f89c06f by human maintainer Andrew Burke (andrew@codepath.org) dated 2026-09-16, 12 days before today"},
    {"name": "Recent use signal", "grade": "pass", "evidence": "Repo pushed_at 2026-09-16T21:48:27Z, 12 days ago, within the 180-day push threshold"},
    {"name": "Bounded contribution", "grade": "pass", "evidence": "Single fix outcome (handle heading-less documents in StructuralChunker) with a named failing test already encoding the expected behavior; no design debate or core-redesign requirement; issue is 18 days old"},
    {"name": "Available work", "grade": "pass", "evidence": "assignees: [] and PR search for the repo plus issue number 56 returned total_count 0; student claim comments do not block per the Path Review house rule"},
    {"name": "AI-assisted workflow allowed", "grade": "pass", "evidence": "docs/CONTRIBUTING.md fetched in full contains no AI ban, and no AI_POLICY.md, AI_USAGE_POLICY.md, or AGENTS.md exists at root or in .github/"},
    {"name": "Verification anchor", "grade": "pass", "evidence": "Issue body includes an executable repro snippet and names the regression test tests/unit/test_structural_chunker.py::test_document_with_no_headings"}
  ],
  "verdict": "accept"
}
```

---

## Eval iterations

**Run history**

All runs used the official harness, its pinned Sonnet model, and the same installed rubric path: `~/.claude/skills/issue-select/rubric.md`. These are the actual runs, in order, on September 28, 2026:

1. First full run, rubric v1: `agreement: 17/20 scored items  (bar: 18/20: below the bar)`. It rejected `issue-01`, `issue-09`, and `issue-19`, whose gold labels are all `accept`. I revised two checks: multi-file work and multiple diagnosed causes no longer imply an umbrella issue, and an old maintainer endorsement no longer keeps a claim alive indefinitely.
2. Partial v2 diagnostic using `--only issue-01,issue-09,issue-19`: `agreement: 3/3 scored items`. All three changed to `accept`. This was not a full passing run and is not the submitted transcript.
3. Final full v2 run using `--save-run eval-run.txt`: `agreement: 19/20 scored items  (bar: 18/20: PASS)`. Its category line is `categories: claimed 4/4  clear-accept 8/8  dead-repo 3/3  policy 1/1  scope 3/4`. The remaining disagreement is `issue-20`. I kept the rubric unchanged after this run and copied the complete harness-generated file without editing it.

I read the four calibration bundles before writing v1; I did not count that reading as an evaluation run. No calibration-only or smoke run occurred.

After calibration, my first live attempt read the installed skill but stopped because CLI help commands were denied by its tool allowlist. It produced no verdict. With those read-only tool permissions corrected, the actual skill graded three open candidates: it ranked https://github.com/codepath/pathreview-ai301-fa26-s1/issues/56 first and https://github.com/codepath/pathreview-ai301-fa26-s1/issues/40 second among accepts, and rejected https://github.com/codepath/pathreview-ai301-fa26-s1/issues/68 because of the open implementation at https://github.com/codepath/pathreview-ai301-fa26-s1/pull/78. I then ran a standalone confirmation on the selected issue, producing the output above. None of these live runs changed the installed rubric or the saved eval artifact.

**Issue analysis**

In the final scored run, `issue-20` has rubric verdict `accept` and gold verdict `reject`. The transcript records:

```text
issue-20  reject  accept   NO     graded accept
```

The frozen bundle asks: "Add a new toolbar shape that inserts a fixed company logo (SVG/image). Users should be able to place, resize, and move it like other elements, and it should export correctly." My check treated that as one defined behavior, rather than requiring a particular implementation. The harness's per-check result says: "One outcome scoped" and "no maintainer blocker stated."

But the same bundle says "Logo asset TBD." There are no comments settling the product decision. The gold note calls out "a product decision hiding inside" the request. I understand that rejection: a concrete interaction description does not prove the project wants this feature or has chosen the asset. My rubric accepted it because it looked for an explicit blocker rather than requiring affirmative product approval. This is a false accept against the gold label, not a perfect result.

**Check rationale**

This is the exact current check uploaded in `tools/issue-select/rubric.md`:

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| Bounded contribution | Issue body, full available comment thread, creation date, and closed-unmerged attempt history; live issue and Development sidebar plus thread references. | There is one identifiable contribution outcome and zero explicit blockers: an umbrella/tracking issue intended to split into independent contributions; an unresolved product/design debate blocking the requested behavior; or a maintainer explicitly requiring a core parser/compiler or architectural redesign. Do not infer these blockers from file count, a checklist, optional suggestions, or multiple diagnosed causes of the same bug. A docs page with supporting page updates is one outcome; a maintainer-diagnosed bug need not prescribe one implementation algorithm. Reject pure usage/support questions or a wish with no defined behavior. Also reject an issue older than 730 days with at least 2 abandoned implementation attempts. A short body, missing repro, or absent good-first-issue label does not by itself fail this check. | required |

I made the evidence source the issue body and thread, not the label. The threshold is one outcome with zero explicit scope blockers; the historical warning needs both more than 730 days and at least two abandoned attempts. Age alone should not reject a valid request. This wording fixes the specific mistake in my first run: a docs page plus supporting edits can be one contribution, and two diagnosed causes can still describe one bug. The final run accepted both `issue-01` and `issue-19`, matching their gold labels.

**Trade-offs**

The phrase "zero explicit blockers" avoids treating every implementation choice as an unresolved design debate. It also misses some requests that need product approval even when nobody has posted an objection. `issue-20` is the concrete cost: the final rubric accepted a request whose bundle still says "Logo asset TBD." I kept that limitation visible instead of claiming the passing score proved the check was complete. Before implementing that kind of feature, I would still want the product and asset questions resolved.

---

## Selection rationale

**Selection rationale**

1. **Fit and time.** Python and TypeScript are my strongest languages. I chose the Python chunking bug because it has one observable failure: documents without headings disappear from the index. The issue provides an executable example and an existing test target. That keeps the first contribution's time risk lower than adding the sharing feature across frontend and backend files. I have not measured implementation time or run the reproduction yet, so I am not promising an hour estimate.
2. **What the verdict got right, and what I weighed.** The skill correctly identified the bounded behavior, recent human activity, available issue, and concrete verification target. It ranked this candidate first. I also weighed the cost of deciding new behavior: preventing silent document loss is a narrower starting point than introducing public-link access and expiration. The rubric does not tell me whether the best fallback is one chunk or another strategy, or whether the existing tests cover every edge case. Those still need investigation in Unit 2. In the comparative run it also overlooked the sharing issue's stated expiration criterion when grading its preferred verification check; that did not change either issue's required verdict.
3. **Claiming.** I expect little difficulty under the classroom rule, even though classmates have already posted claims. The installed scope explicitly says their claim comments "do not block an issue." The standalone run found no assignee or open implementation for this issue. I have selected it only: I have not commented, claimed, assigned, implemented, or opened a contribution pull request.

Related paths: `eval-run.txt` in this directory; the installed skill copy in `tools/issue-select/`.
