# Review checklist

Used by the reviewer routine and by Mohannad at milestone review.

## Every pull request

- [ ] Targets a `milestone/` branch, never main
- [ ] Tests and lint pass
- [ ] Every acceptance criterion in the issue is met and has a test
- [ ] Follows docs/architecture.md and docs/data-model.md
- [ ] Respects every entry in docs/decisions.md
- [ ] Stays within the issue's scope
- [ ] No secrets, keys or passwords in the code
- [ ] No co-author or "generated with" lines in commits

## Milestone review (gate 2, Mohannad)

Shortcut: `/finish-milestone <slug>` in an ordinary `claude` session runs these checks, writes a hand-test script, and drafts the pull request body. You still open and merge the pull request yourself.

- [ ] All milestone issues closed
- [ ] The latest CI run on `milestone/<slug>` passed (it runs after every merge into the branch, so it tests the combined code)
- [ ] Demo works on the milestone branch: run it for real and click through the screens with a written script; check the database too. Do not skip this, tests alone have missed a crash on first start
- [ ] Skim changes under `.github/`, auth code and the example env file (no real secrets)
- [ ] Say plainly in the pull request body anything you did not test by hand
- [ ] Read the list of design decisions made during the milestone, then have the design partner archive them ("milestone close"), so docs/decisions.md keeps only one-line rules
- [ ] Open a pull request from `milestone/<slug>` into main and check that `test` and `authorship` pass
- [ ] Merge it yourself with Squash and merge (message = pull request title). Before confirming, read the commit message box and delete any `Co-authored-by` line
- [ ] Afterwards confirm every commit on main is yours with no co-author lines: `git log origin/main --format="%h %an <%ae> | %(trailers:key=Co-authored-by)"` shows only you, with nothing after the `|`. The `authorship` run on the push to main passes too

## Deploy (optional gate 3, Mohannad)

Only if the app is deployed somewhere.

- [ ] It looks right in wherever it is tried first (a staging copy, if you have one)
- [ ] Deploy only after you say so
