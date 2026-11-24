---
name: new-milestone
description: Creates milestone/<slug> from the latest main with the design docs committed on it, after checking the feature doc has numbered acceptance criteria. Run only when Mohannad asks, in an ordinary claude session (not design.ps1).
argument-hint: <slug>
disable-model-invocation: true
---

You are helping Mohannad start a milestone. The slug is: $ARGUMENTS

This replaces the six manual commands in workflow/README.md, step 2. You only touch git locally and push one new milestone branch. You never push to main, never force push, and never commit anything outside `docs/` and `CLAUDE.md`.

If you cannot run shell commands (for example this is a design partner session), stop and tell him to exit and run `claude` in the repo root instead.

## 1. Check (read-only), and stop at the first problem

1. Slug: if it is empty, ask for one. It must be lowercase letters, digits and hyphens only, for example `user-login`. If it isn't, suggest a fixed version and wait.
2. You are at the repo root (`git rev-parse --show-toplevel`) and `origin` exists.
3. The branch does not exist yet: `git ls-remote --heads origin milestone/<slug>` prints nothing, and `git branch --list milestone/<slug>` prints nothing. If it exists, stop: the milestone was already started.
4. Uncommitted changes: `git status --porcelain`. Every changed or new path must be under `docs/` or be `CLAUDE.md`. If anything else is changed, list it and stop; he decides what to do with it.
5. There is something to commit: at least one changed path under `docs/`. If nothing changed, stop and tell him to run the design session first (`powershell -ExecutionPolicy Bypass -File design.ps1`).
6. Feature doc: find the new or changed file(s) under `docs/features/` (not README.md). There must be one. Read it and check it has a line starting with `Acceptance criteria` followed by numbered lines (`1.`, `2.`, or lettered ones like `19b.`). If there is none, or no numbered criteria, stop and tell him to finish it with the design partner.
7. Whether real CI is set up yet: `.github/workflows/ci.yml` on main still contains the kit's placeholder step if it includes the text `CI is still the placeholder`. Remember the answer for step 4. (Don't judge this by commit messages: a project that adopted the kit late can have real CI before its first `Milestone` merge.)
8. Protection on main: `gh api "repos/{owner}/{repo}/rules/branches/main" --jq '[.[].type] | join(" ")'` must include `pull_request` and `non_fast_forward`. If it prints nothing or misses either, stop: main is not protected, so a thread could push straight to it. Point him to SETUP.md, section 3 (`protect-main`); on a private repo this needs GitHub Pro.
9. Protection on milestone branches: run the same command for `milestone/<slug>`. If `non_fast_forward` is missing, warn that the `protect-milestones` ruleset isn't set up (SETUP.md, section 3), then carry on.

## 2. Show the plan and wait

Show him, in plain words:
- the branch: `milestone/<slug>`, created from the latest main
- the files that will be committed (from step 1.4)
- the feature doc and how many acceptance criteria it has
- the commit message: `Design docs for <slug>`

Ask "Go ahead?" and wait for a clear yes. Pushing creates the branch on GitHub.

## 3. Create, commit, push

Run these one at a time, and stop and explain at the first error (do not retry with different flags):

```
git switch main
git pull --ff-only
git switch -c milestone/<slug>
git add docs CLAUDE.md
git commit -m "Design docs for <slug>"
git push -u origin milestone/<slug>
```

- `git switch main` carries the uncommitted docs along. If git refuses because they would be overwritten, stop and explain; nothing has been changed yet.
- If `git pull --ff-only` fails, main has diverged locally. Stop; never reset or force anything.
- Do not add co-author or "generated with" lines. The repo's `.claude/settings.json` already keeps them out.

## 4. Verify and hand over

1. `git ls-remote --heads origin milestone/<slug>` shows the branch.
2. `git log -1 --format="%an <%ae> | %(trailers:key=Co-authored-by)"` shows Mohannad, with nothing after the `|`. If there is a co-author line, tell him straight away and do not try to rewrite the commit.
3. Tell him the next step, with the exact text to paste:
   - CI still the placeholder (from step 1.7): the real CI comes before any thread. See SETUP.md, section 7, step 3.
   - Otherwise: paste into the Claude Code Project: `Plan docs/features/<file>.md on milestone/<slug>.`
