# How the workflow runs

Mohannad is the architect. Agents do most of the building. Mohannad signs off at two gates: plan and milestone. If the app gets deployed somewhere, he also signs off on each deploy (an optional third gate).

The manager and workers are Claude Code Projects: the project conversation is the coordinator (manager), and each thread is a worker running as a cloud session. Cloud threads read CLAUDE.md but not .claude/settings.json, hooks or skills, so their rules live in the project instructions (workflow/project-instructions.md) and in CLAUDE.md. The real protection for `main` is the GitHub ruleset, not the instructions.

## The loop for one feature

1. Design (you + design partner, local): start a design session with `powershell -ExecutionPolicy Bypass -File .\design.ps1`. It writes the feature goal with numbered acceptance criteria in docs/features/.
2. Milestone branch (you): create it locally from the latest main and push the docs to it:
   `git switch main`, `git pull`, `git switch -c milestone/<slug>`, `git add docs CLAUDE.md`, `git commit -m "Design docs for <slug>"`, `git push -u origin milestone/<slug>`.
   (First milestone only: also put the real CI on this branch before any thread starts; see SETUP.md section 7, step 3.)
3. Plan (coordinator): in the project conversation, say "Plan docs/features/<file>.md on milestone/<slug>." It proposes threads in a Suggested threads list.
4. Gate 1 (you): ask for changes until the plan is right, then click the arrow on each suggestion, or the button that starts all of them. At most 2 at a time; threads touching the same files go one after the other.
5. Build (threads, cloud): each thread branches from the milestone branch and opens one normal (not draft) pull request into it. They keep going with your laptop closed. Track them in the Overview pane.
6. Review (reviewer routines, cloud): one fires when a pull request is opened and a second when commits are pushed; they set exactly one of `reviewer-approved`, `changes-requested`, or `needs-decision`. Threads are supposed to fix the reviewer's comments on their own, but often go idle instead: if a thread has done nothing after `changes-requested`, paste "TN: address the reviewer's numbered fixes on PR #N: ..." (with the fixes) into the project.
7. Merge into the milestone (GitHub Actions): a pull request into a `milestone/` branch is squash merged automatically once it is labelled `reviewer-approved` and CI has passed on the same commit. A new push removes the label until it is re-reviewed. After each merge, CI runs on the milestone branch to test the combined code.
8. Decisions: a `needs-decision` thread shows under Waiting on you. Open a design session, answer it, commit and push the updated docs to the milestone branch, and reply in that thread. The thread builds the answer, removes the label and pushes, which starts the re-review.
9. Gate 2 (you): when the milestone is done, check the latest CI run on the milestone branch passed, hand-test it (run the app and click through it), follow the milestone section of workflow/review-checklist.md, then squash merge `milestone/<slug>` into main yourself, reading the commit message box first and deleting any `Co-authored-by` line. With the ruleset on, the PR shows BLOCKED until the `test` and `authorship` checks pass, then CLEAN. `authorship` fails if any commit isn't yours or credits a co-author.
10. Optional gate 3 (you): only if the app is deployed somewhere. Deploy when you are satisfied with how it looks wherever it is tried first (a staging copy, if you have one).

## Labels

| Label | Meaning |
| --- | --- |
| `reviewer-approved` | Passed review; the merge workflow merges it once CI passes. Removed automatically by any new push |
| `changes-requested` | Reviewer found fixable problems |
| `needs-decision` | A design question only Mohannad can answer; the reviewer skips the PR until it is removed |
| `ticket`, `ready`, `in-progress` | Only used by the fallback manager commands |

Only the reviewer sets `reviewer-approved` and `changes-requested`. Threads never do.

## Watching progress

- The project's Overview pane: Ready for review, Waiting on you, Working
- Desktop notifications from the Claude desktop app (signed in to your own account)
- GitHub mobile app for pull requests
- The repo's Actions tab: "Merge reviewed PRs" says in its log why a pull request was or wasn't merged

## Fallback

If Projects is ever unavailable, the local manager commands still work: `/plan-feature` and `/dispatch-workers` in a terminal Claude Code session, with workers following workflow/worker.md.
