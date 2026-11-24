# Setup checklist

Kit v4.1. Do these once, in order. Everything runs on your own Claude account (Pro or Max), never a shared one.

## 1. Tools on your computer (Windows, in PowerShell)

- [ ] Git for Windows (includes Git Bash, which Claude Code uses for shell commands): `winget install Git.Git`
- [ ] Node.js (needed for the design partner's guard script): `winget install OpenJS.NodeJS.LTS`
- [ ] GitHub CLI: `winget install GitHub.cli`, then close and reopen PowerShell and run `gh auth login` with your personal GitHub account
- [ ] Only if the app will run in containers: Docker Desktop. Use the "All users" installer (it needs admin to enable Virtual Machine Platform), run `wsl --update`, and make sure virtualization is on in the BIOS (AMD: SVM Mode, Intel: VT-x). Check with `(Get-CimInstance Win32_Processor).VirtualizationFirmwareEnabled` (should say True), then `docker run --rm hello-world`. Docker Desktop does not start itself after a restart; open it before using `docker` commands.
- [ ] Claude Code: `irm https://claude.ai/install.ps1 | iex`, reopen PowerShell, run `claude --version` to check, then `claude` and log in with your own Claude account. Run `/status` and check it shows your subscription (Pro or Max), not an API key. If it still shows an old plan after you upgrade, run `/logout` and `/login`.

If PowerShell refuses to run design.ps1 ("running scripts is disabled"), start it with `powershell -ExecutionPolicy Bypass -File .\design.ps1` instead.

## 2. The repo

- [ ] Check which GitHub plan the account is on, because it decides whether `main` can really be protected (section 3).
  - GitHub Pro, including the free student Pro from the Student Developer Pack: rulesets work on private repos. Nothing to do. (The student benefit has to be renewed; if it lapses the account drops to Free and private-repo rulesets silently stop being enforced.)
  - GitHub Free: rulesets and branch protection are NOT enforced on private repos (the API answers 403 "Upgrade to GitHub Pro or make this repository public"). Choose one: make the repo public (only if nothing in it is secret), upgrade, or accept that `main` is protected only by the project instructions and check `git ls-remote --heads origin main` before each wave.
- [ ] Create the GitHub repo for the app (optional: try the workflow on a throwaway sandbox repo first; your first real milestone also works as the test).
- [ ] In the project folder, run `git init -b main` (without it, `git config` says "not in a git directory"), then set your identity for this repo: `git config user.name "Your Name"` and `git config user.email "you@example.com"` (use the name and email on your GitHub account).
- [ ] Copy everything in this kit into the repo root, commit, and push to main. This is the only time you push straight to main; once the ruleset in section 3 is on, main only changes when you merge a milestone.
- [ ] From the repo root, run `bash workflow/setup-labels.sh` to create the labels.
- [ ] Make sure the Claude GitHub App has access to this repo (github.com, Settings, Applications, Claude, Configure).

## 3. GitHub settings (Settings tab of the repo)

- [ ] General, Pull Requests: allow squash merging only, set "Default commit message" for squash merging to "Pull request title" (the commit gets the PR title and no body by default; GitHub still pre-fills `Co-authored-by` lines in the merge box when a commit on the branch has them, so read that box whenever you merge by hand), and turn on "Automatically delete head branches". (After a merge, clean up your own copy with `git fetch --prune`, then `git branch -D` any local branch marked `[gone]`.)
- [ ] Actions, General, Workflow permissions: "Read and write permissions".
- [ ] Rules, Rulesets, New branch ruleset, named `protect-main`:
  - Enforcement: Active
  - Bypass list: leave empty, so the rule applies to you and the agents alike (the agents act as your GitHub account, so a bypass for you is a bypass for them)
  - Target: the default branch
  - Turn on: Restrict deletions, Require a pull request before merging (0 approvals, since you can't approve your own PRs), Require status checks to pass (add `test` and `authorship` once each has run: Add checks, choose them from GitHub Actions; `authorship` first runs on your first push to main), Block force pushes
  - Check it is really enforced (needs a plan that supports it, see section 2): `gh api repos/OWNER/REPO/rules/branches/main` should list the rules.
- [ ] A second branch ruleset, named `protect-milestones`:
  - Enforcement: Active, bypass list empty
  - Target: Add target, Include by pattern, `milestone/**`
  - Turn on only: Block force pushes
  - Leave the rest off on purpose: "Require a pull request" or "Require status checks" would block your own pushes of docs to the milestone branch, and "Restrict deletions" would stop GitHub deleting the milestone branch after you merge it into main.
- [ ] Projects: create a board and link it to the repo.

The CI workflow must stay named `CI` with its job named `test`; the merge workflow and the required check both use those names. The `Authorship` workflow (job `authorship`) blocks any pull request into main that has a commit not authored by you or a co-author line. If you later add a deploy job to CI: merges made by the merge workflow don't start `push` runs, so a deploy job that only runs on `push` skips every auto-merged pull request. Include `workflow_dispatch` in its `if:` (the merge workflow starts CI that way after each merge) when every merge should deploy, for example to staging.

## 4. Claude Code Projects

- [ ] Signed in to your own Claude account, go to claude.ai/code (or the Code tab of the desktop app), open Projects, and click New project.
- [ ] Name it after the app, leave Goal empty for now, and under Context add only this repository. Click Create project. (On a first project Claude may start an exploring thread or post Setup recommendations on its own; you can ignore or switch them off.)
- [ ] Open Project settings (gear icon). In General set Thread model to Sonnet 5.5 and Thread effort to Medium.
- [ ] In Memory > Project instructions, paste the text from workflow/project-instructions.md (fill in the two placeholders) and save.

## 5. Reviewer routines (two)

- [ ] Follow workflow/reviewer-routine.md to create both routines at claude.ai/code/routines: `PR reviewer` (trigger: Opened) and `PR reviewer (re-review)` (trigger: Commits pushed).

## 6. Notifications

- [ ] Install the GitHub mobile app and sign in with your account. Watch the repo (Custom: Pull requests, Issues).

## 7. The first milestone (a sandbox repo, or your first real one)

Every milestone follows the same steps; workflow/README.md has the short version.

1. [ ] Design: `powershell -ExecutionPolicy Bypass -File .\design.ps1` starts the design partner. On the first milestone it fills in CLAUDE.md and the docs; every milestone, it writes the feature goal in `docs/features/` with numbered acceptance criteria. (For a sandbox test, say so and pick a tiny feature, for example "a function that adds two numbers, with tests".)
2. [ ] Milestone branch with the docs on it. From the repo root:
   ```
   git switch main
   git pull
   git switch -c milestone/<slug>
   git add docs CLAUDE.md
   git commit -m "Design docs for <slug>"
   git push -u origin milestone/<slug>
   ```
   Do not push the docs to main (the ruleset blocks it) and do not create the branch on GitHub from main (it would not have the docs).
   Shortcut: run `/new-milestone <slug>` in an ordinary `claude` session (not design.ps1). It checks the docs, shows you the plan, and runs these commands once you say yes.
3. [ ] Real CI, first milestone only, before any thread starts. The design partner cannot touch `.github/` and threads must not, so do it in an ordinary Claude Code session (`claude` in the repo, not design.ps1): ask it to replace the placeholder step in `.github/workflows/ci.yml` with the install, lint, type check and test commands from CLAUDE.md, the database tests if the app has a database, plus a step that starts the app for real and checks it answers (an example is in the file's comments). Read the change, then commit and push it to the milestone branch yourself. Once it has run, add `test` as the required check in `protect-main`. A sandbox test can keep the placeholder.
4. [ ] Plan: in the project, say "Plan docs/features/<file>.md on milestone/<slug>." It proposes threads and waits for you. Ask for changes until it is right, then start them.

The first time through, confirm each mechanism works:

- [ ] In design mode, asking it to edit a file outside docs/ and CLAUDE.md is blocked by the guard.
- [ ] Started threads branch from the milestone branch and open normal (not draft) pull requests into it, not main.
- [ ] The reviewer routine fires, comments starting with "Reviewer:", and adds one label. If it can't add labels, tell Claude; we'll switch it to a comment keyword.
- [ ] On changes-requested, the thread fixes the comments on its own. If it just goes idle (this happens), paste into the project: "TN: address the reviewer's numbered fixes on PR #N: ..." and restate the fixes.
- [ ] A push to a pull request that already has `reviewer-approved` removes the label (Actions tab: "Merge reviewed PRs", job `dismiss-stale-approval`), and the re-review adds it back.
- [ ] The merge workflow squash merges only once the PR is approved and CI passed on the same commit, and only into the milestone branch. Right after, a CI run starts on `milestone/<slug>` (Actions tab).
- [ ] Reviewer comments start with `Reviewer: approved at <sha>` (or `changes-requested at` / `needs-decision at`). A commit that has a changes-requested or needs-decision review is never merged, even if a second review of it approves.
- [ ] The commits on the milestone branch show you as the author with no Claude co-author line.
- [ ] The `authorship` check passes on the milestone pull request into main. You merge it yourself with squash, after reading the commit message box and deleting any `Co-authored-by` line. Afterwards `git log origin/main --format="%h %an <%ae> | %(trailers:key=Co-authored-by)"` shows only you, with nothing after the `|`.
- [ ] Project settings > Usage shows how much the test used, so you can judge your plan's limits.

When all boxes are ticked, start the real app with a project design session.

## 8. Habits that paid off

- **Hand-test every milestone before merging it to main.** Run the app for real, click through the screens with a written script, and check the stored data. CI's startup check catches crashes on start, but not wrong behaviour. Stop the app afterwards.
- **Check every recommendation for its worst case.** Ask what real thing it would hide or break, and check against real data when you can, not just reasoning.
- **Never paste secrets into a chat or a pull request.** Edit files that hold secrets yourself. If a secret is pasted anywhere, rotate it.
- **Run at most 3 threads at once.** Threads that edit the same files go one after the other, so 3 only helps when the work splits cleanly.
- **Only on GitHub Free:** run `git ls-remote --heads origin main` before each wave and check the commit has not moved. With the ruleset enforced, GitHub does this for you.
- **Empty the merge box when you merge by hand.** For a pull request that changes `.github/` (the merge workflow refuses those) and for a milestone into main, GitHub pre-fills the "Extended description" box with `Co-authored-by` lines from the branch's commits, whatever your squash settings say. Empty it before **Confirm squash and merge**. The merge workflow blanks it for you; a hand merge doesn't. If a line slips into a milestone branch, the `authorship` check fails at gate 2.
- **Merged branches vanish on GitHub** (auto-delete), so "remote ref does not exist" when you try to delete one is normal.
