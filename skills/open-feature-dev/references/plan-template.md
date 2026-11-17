# Plan template (Step 5)

Write `$D/plan.md` in this shape:

```markdown
# Plan: <title> — <issue url or spec path>
## Goal
## Decisions            (verbatim from the clarify gate; binding)
## Approach             (chosen; each rejected alternative with the reason)
## Non-goals
## Global constraints   (repo rules and exact values every task inherits)
## Tasks
### Task 1: <name>
- Files: create `path`; modify `path:12-40`
- Behaviour: <what it does, in terms of inputs and outcomes>
- Interfaces: consumes `fn(a: T) -> U` from Task N; produces `…` for Task M
- Test first: `tests/test_x.py::test_case` asserts <…>   (or `none — <why>` for a reversible, low-impact change whose test would only restate the code: copy, a config value no code branches on, docs)
- Done when: <observable condition>
- Task complexity: <low | medium | high> → <fast | balanced | strongest> — <the reading that set it: files, contract, or ambiguity>
## Review focus         (≤ 5 input classes or failure modes the tasks' tests do not cover, each assigned to a task)
## Verification         (end-to-end commands that prove the feature, with expected output)
## Complexity tracking
| Addition | Why needed | Simpler alternative rejected because |
## Rollout and risk     (migration, flags, compatibility window, how to revert)
```

A task is the smallest unit a reviewer could reject while approving its neighbour: one to three files, one testable outcome. Same-shape edits across many files are one task. The plan is decisions, not code; a step that says "add appropriate error handling", "similar to Task N", "TBD", or names a type no task defines is a plan failure.

Self-review before dispatch: every decision and acceptance criterion maps to a task; placeholder scan; the same name and signature for a thing wherever tasks mention it; every review-focus line is assigned; the Complexity table is empty or every row is justified; every behaviour change a user can see has the document that describes it in some task's Files (grep the README, `docs/`, help text, and changelog for the old behaviour), or Non-goals says `Docs: none describe this`; every task has a Task complexity line, and none on a risk surface or a public API is low.
