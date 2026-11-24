---
name: finish-milestone
description: Gate 2 helper. Runs every check before Mohannad merges milestone/<slug> into main (CI, authorship, secrets, .github changes, open PRs), helps him hand-test, and writes the pull request body. Read-only; never opens, merges or pushes. Run only when he asks.
argument-hint: <slug>
disable-model-invocation: true
---

You are helping Mohannad with gate 2: merging a finished milestone into main. The slug is: $ARGUMENTS (if empty, use the current branch when it is `milestone/<something>`, otherwise ask).

You are read-only. You never open the pull request, never merge, never push, never edit files in the repo, and never print a secret's value. `main` is his alone; you prepare everything and he runs the last command himself.

Run commands in Bash (Git Bash), from the repo root. If `gh auth status` fails, stop and tell him to run `gh auth login`.

## 1. Checks

Start with `git fetch --prune origin`. Let `M=origin/milestone/<slug>`. If `git rev-parse --verify $M` fails, stop: the branch doesn't exist on GitHub.

Run every check, even after a failure, then report them all together as a list marked PASS, FAIL or CHECK (CHECK = needs his eyes, not necessarily wrong).

1. **Something to merge:** `git rev-list --count origin/main..$M` is above 0.
2. **No work still open:**
   - `gh pr list --base milestone/<slug> --state open --json number,title,labels`: should be empty. List any that aren't, with their labels.
   - `gh issue list --milestone "<slug>" --state open --json number,title`: should be empty (an error saying the milestone doesn't exist counts as empty).
3. **CI passed on the combined code:** `gh run list --workflow CI --branch milestone/<slug> --limit 1 --json status,conclusion,headSha,url`. PASS only if it is completed, `success`, and `headSha` equals `git rev-parse $M`. If the latest run is for an older commit or still running, say so. If there is no run for the head commit, offer to start one with `gh workflow run ci.yml --ref milestone/<slug>`, and start it only if he says yes.
4. **Authorship** (the same rules as `.github/workflows/authorship.yml`):
   - `git log origin/main..$M --format="%h %an <%ae> | %(trailers:key=Co-authored-by)"`: every line should show only him, with nothing after the `|`.
   - Also run:
     ```
     OWNER=$(gh repo view --json owner -q .owner.login) gh api "repos/{owner}/{repo}/compare/main...milestone/<slug>" --jq '.commits[] | .sha[0:7] as $s | (if ((.author.login // "") | ascii_downcase) != ($ENV.OWNER | ascii_downcase) then "\($s): author \(.commit.author.name), GitHub account \(.author.login // "none")" else empty end), (.commit.message | split("\n")[] | select(test("^[[:space:]]*co-authored-by:"; "i")) | "\($s): co-author line"), (if (.commit.message | test("generated with .?claude"; "i")) then "\($s): Generated with Claude line" else empty end)'
     ```
     No output = PASS. Any output = FAIL, and the `authorship` check on the pull request will fail too. Never suggest fixing it by force pushing; explain what you found and let him decide.
5. **Secrets.** Print only file names and line numbers, never the matching text:
   ```
   git grep -nIE -e '-----BEGIN [A-Z ]*PRIVATE KEY-----' -e 'AKIA[0-9A-Z]{16}' -e 'gh[pousr]_[A-Za-z0-9]{30,}' -e 'sk-[A-Za-z0-9_-]{20,}' -e 'xox[baprs]-[A-Za-z0-9-]{10,}' -e 'AIza[0-9A-Za-z_-]{35}' -e 'GOCSPX-[A-Za-z0-9_-]{10,}' $M -- . | cut -d: -f2-3
   ```
   ```
   git grep -nIiE "(password|passwd|secret|token|api_?key)[\"']?[[:space:]]*[:=][[:space:]]*[\"'][^\"']{8,}[\"']" $M -- . | cut -d: -f2-3
   ```
   Also list any file in `git diff --name-only origin/main...$M` named `.env` or `.env.*` (except `.env.example`), `*.pem`, `*.key` or `id_rsa*`.
   Matches inside tests can be fake keys: mark those CHECK, not FAIL, and say which file. Real-looking ones are FAIL; tell him to remove the secret and rotate it at the provider.
6. **The example env file:** if `.env.example` changed (`git diff --name-only origin/main...$M -- .env.example`), list its variable names and, for each, whether the value is empty, an obvious placeholder, or something that could be real. Never print the values.
7. **Changes under `.github/`:** `git diff --stat origin/main...$M -- .github/`. Mark CHECK if any, and list them for him to skim (he merged those by hand, but they deserve a second look).
8. **Summary of the change:** `git diff --stat origin/main...$M | tail -1`, plus the merged pull requests: `gh pr list --base milestone/<slug> --state merged --limit 100 --json number,title`.
9. **Protection on main:** `gh api "repos/{owner}/{repo}/rules/branches/main" --jq '[.[].type] | join(" ")'` includes `pull_request`, `non_fast_forward` and `required_status_checks`, and `gh api "repos/{owner}/{repo}/rules/branches/main" --jq '[.[] | select(.type == "required_status_checks") | .parameters.required_status_checks[].context] | join(" ")'` includes both `test` and `authorship`. FAIL if anything is missing: the pull request could then be merged without those checks. Point him to SETUP.md, section 3.
10. **Decisions archived:** on `$M`, `docs/decisions.md` should have no full entries left under "Full entries for the current milestone" (the format example inside the code block doesn't count). If some are left, mark CHECK and suggest a design session first: `powershell -ExecutionPolicy Bypass -File design.ps1`, then "milestone close for <slug>". He commits and pushes the docs to the milestone branch, waits for CI, then runs this skill again.

## 2. Hand-test

Ask whether he has hand-tested the milestone (run the app for real, click through it, check the stored data). Tests use fakes; a setup that was never started has passed every test and still crashed before.

- If not yet: read the feature doc(s) changed in this milestone (`git diff --name-only origin/main...$M -- docs/features/`) and write a numbered click-through script from the acceptance criteria, each step with its expected result. Remind him to stop the app afterwards. Then wait until he reports back.
- Ask which criteria he tested by hand and which he didn't. Both go in the pull request body.

Do not write the body until every check is PASS or he has explicitly accepted each FAIL and CHECK.

## 3. The pull request body and the command

1. Milestone number N = 1 + the number of subjects on main starting with `Milestone ` (`git log origin/main --format=%s`). Name = the feature doc's title.
2. Write the body to a file in the system temp folder (never inside the repo), for example `milestone-<slug>-pr.md`. Contents:
   - one or two sentences on what the milestone does
   - the merged pull requests (number and title)
   - the acceptance criteria: tested by hand, and not tested by hand (covered by automated tests only)
   - the checks from section 1 and their results
   - no co-author lines, no "Generated with" lines, no session links
3. Give him the command to run himself in PowerShell, with the real values and the file's full Windows path:
   ```
   gh pr create --base main --head milestone/<slug> --title "Milestone N: <name>" --body-file "<full path>"
   ```
4. Then remind him, as a short numbered list:
   1. Wait until `test` and `authorship` both pass (`gh pr checks`). The PR shows BLOCKED until then.
   2. Merge with **Squash and merge**, title as the message. **Before confirming, read the message box and delete any `Co-authored-by` line.**
   3. Afterwards, check main: `git fetch` then `git log origin/main --format="%h %an <%ae> | %(trailers:key=Co-authored-by)"` shows only him, with nothing after the `|`.
   4. Clean up: `git fetch --prune`, then `git branch -D` the local branches marked `[gone]` (`git branch -vv` shows them).
   5. Run `/milestone-retro <slug>` (a personal skill; skip if it isn't installed). It files a short workflow retro so the kit can improve; its questions are optional.
