# PR/MR body, push commands, and final report (Steps 10 and 11)

## Title and body

Title: imperative, at most 70 characters, the repo's prefix convention if it has one. Body to `$D/pr.md`, using the repo's template as the skeleton when there is one (its headings, its ticket-link line; not its habit of listing every file). Otherwise:

```markdown
## Why
<the problem, restated so the PR stands alone without the issue; then, when the issue lives on this forge> Closes #<N>
## What changed
<two to five bullets of outcomes and decisions — what the diff cannot show; not a file list>
## How this was verified
<each command and its result; tests added by behaviour, not by count; then what was not exercised: a path or case no test or command reached>
## Risks and rollout
<blast radius, migration, flag, what to watch, how to revert>
## Known limitations and follow-ups
<findings left open, in plain words; out-of-scope work noticed>
## Decisions made without asking
<each Ruling in plain words, with its cost if wrong>
```

One closing keyword per issue (`Closes #12, closes #13`; cross-repo `Fixes owner/repo#12`); warn the user when the base is not the default branch, because the keyword is ignored there. In the skeleton above, delete a section that would be empty rather than writing "N/A". In a repo's own template, keep every heading and checklist, and write `N/A` under one that does not apply. A PR reader has not seen this skill, so the body uses none of its labels. AI attribution follows the repo: add it when the contribution rules or the PR template ask for it or recent PRs carry it, and not otherwise.

## Commands, after the user's yes at the push gate

```bash
git push -u origin HEAD
gh pr create --title "<title>" --body-file $D/pr.md --base <base> [--draft] [--label …] [--reviewer …]
gh pr view --json url,closingIssuesReferences        # the link registered, or it did not
glab mr create --title "<title>" --description-file $D/pr.md -b <base> --related-issue <N> --yes [--draft]
glab mr view                                         # probe flags with --help first; older glab wants --description "$(cat $D/pr.md)"
```

Neither CLI: on GitLab, `git push -u origin HEAD -o merge_request.create -o merge_request.target=<base> -o merge_request.title="<title>"` creates the MR from the push; otherwise push, then print the compare URL (`https://github.com/<owner>/<repo>/compare/<base>...<branch>?expand=1`, or `https://<host>/<group>/<project>/-/merge_requests/new?merge_request%5Bsource_branch%5D=<branch>&merge_request%5Btarget_branch%5D=<base>`) and the body in a fenced block for the user to paste; a remote that is not a forge gets the branch name and the body only.

## Final report (Step 11)

```markdown
## Feature Dev Results

**Issue**: <url> · **Spec**: <path, or "none: <why>"> · **PR/MR**: <url, or "not created: <why>"> · **Tier**: direct | full · **Branch**: <name> → <base>
**Tasks**: N · **Review rounds**: R · **Findings**: A fixed, B dismissed, C deferred, D parked
**Task complexity**: Task 1 low/fast → 0 rounds · Task 2 medium/balanced → 4 rounds, fresh implementer · models: <the ledger's `models:` line>

### What was built
### Verification
- `<command>` → <result>
### Rulings — decisions made without asking you
- <what> — <why> — costs <…> if wrong
### Dismissed — findings disproved
- <finding> — disproved by `path:line`: `<quoted line>`
### Parked and deferred — parked: still open when the round cap was reached; deferred: real, left for later because it is low severity or outside this change
### Follow-ups and out of scope
### Coverage
- spec: open-task-spec (cold reads <n>) | given approved | skipped (<why>) · plan reviewer: subagent | in-line | missing (<why>) · implementer ×N · task reviewer ×N · code reviewer: open-code-review | fallback
- CI: <state> · skipped: <what and why>
```

Every number comes from the ledger or a command you ran; Findings counts cover every review in the run (plan, task, final), excluding a reviewer's own dismissals. The Rulings list is exhaustive. The Task complexity line is full tier only: the tag comes from the plan and the rounds from the ledger, so the user can see whether the tags predicted the work. Leave the workspace in place for PR feedback; the user deletes it when the PR lands.
