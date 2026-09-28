# Evidence guide: where proof lives in a reproduction package

## Environment

- Where it lives: In eval, compare the issue context and thread's target with the repro report's environment record, version output and configuration in its commands. In live mode, compare the draft with the issue and the repository README, setup docs and dependency manifest at the tested revision.
- What good looks like: The platform, software version or commit and trigger-relevant settings/dependencies identify a runnable state, with relevant deviations explained. A version range in installation instructions can be paired with recorded resolved versions; exact versions of unrelated tools are not required and secret values should never be published.

## Steps

- Where it lives: Read the repro report from its stated starting state through installation, fixtures/input, commands or UI actions to the observation. In live mode follow its public documentation links read-only to check prerequisites, not to supply missing private draft evidence; in eval the bundle is the entire available world.
- What good looks like: A stranger can repeat the decisive action with the same substantive input and configuration, without guessing file contents, editing production code or relying on the author's local state. A short executable example or fully specified UI path can suffice; hidden artifacts, an absent trigger, or "run the app and see" cannot.

## Behavior shown

- Where it lives: Extract the initiating action, relevant conditions and visible failure from the issue body and thread, then compare that tuple with the repro's input, command output, logs, screenshots and concrete observations. In live mode read the posted claim plus the proposed repro as the public package; unrelated local logs do not rescue evidence missing from those comments.
- What good looks like: The artifact makes the expected/actual difference observable on the issue's trigger, rather than proving only a similar error on a different path. A control or repeated attempt can strengthen that distinction without being mandatory for every bug; a cannot-reproduce package needs concrete observed nonfailure on the attempted trigger, not merely "works for me."

## Honesty

- Where it lives: Compare the claim's promises and the repro's outcome, scope and explanation with its artifacts and named tested environment; compare any causal assertions with what the issue/thread actually establishes. In live mode also check chronology in the issue comments; in eval do not invent missing history.
- What good looks like: The author says what happened under the tested conditions and keeps inference separate from observation. A cannot-reproduce report can be ready even though the original bug is unresolved; a dependency failure before the trigger is only a setup failure, and confidence or another student's report cannot replace independent proof.

## Comms

- Where it lives: In eval, read the claim against the issue and compare both candidate comments with the repo-facts bug-template and contribution/AI-policy entries. In live mode read the current full issue thread, README-linked contribution docs (including docs/CONTRIBUTING.md), relevant issue template under .github/ISSUE_TEMPLATE, and any linked or root/.github AI-policy file; use gh-axi read-only API calls or public pages. Apply scope.md's classroom rules, and separately report voice-guide.md violations.
- What good looks like: The claim names the specific action/condition and behavior plus a relevant next investigation/report step, without boilerplate blame or invented experience. Comply with explicit policy as written: required AI disclosure must actually appear in the prescribed place with the required details (not just an unrelated acknowledgment elsewhere). Course eval candidates are AI-assisted work, so missing disclosure cannot be excused by assuming no AI was used. A policy allowing AI with human responsibility alone does not demand an explicit declaration. Use bug-template prompts to locate relevant facts; a follow-up repro need not paste every diagnostic dump from an original issue template if its environment and minimal trigger already establish the needed facts. Do not waive explicit rules that actually apply to comments, or invent a ban or disclosure duty from policy silence.

If a live policy fetch fails, try the linked official location or public page once. If essential policy remains inaccessible, mark the relevant check unclear and identify the exact missing source; do not treat an access error as proof of policy silence. A missing optional policy file is not a failure after the linked contribution guide and policy locations have been checked. No Unit 2 worksheet was supplied for these checks; they map the official proof families to observable evidence.
