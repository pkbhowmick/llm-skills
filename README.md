# llm-skills

Skills for Claude Code and Codex. Each directory is one skill (`<name>/SKILL.md`). No dependencies beyond git.

## open-code-review

Exhaustive, line-level code review run entirely by the agent. Whole files are read, not hunks. Every changed unit gets a contract map (inputs and trust, assumptions and the line that establishes each, callers including indirect ones, callees including library calls, removed code, refactor equivalence), then is checked against eleven bug-hunting dimensions (logic, edges, error paths, concurrency, resources, security, data integrity, contracts, performance, tests, and git history — whether the change undoes a past fix), an absence pass for what the diff should contain but does not, and a strict style lens. A skeptic pass can dismiss a finding only with a cited line, and every dismissal is reported. Then a second cold read, a verdict that follows fixed rules, and a per-unit coverage grid. No CLI, no API key.

The hunt runs without a severity floor and the filtering happens afterwards, in one pass with its own rules, because a reviewer told to be conservative is conservative about real bugs too — and a reviewer told to find problems finds them whether or not they exist. So the filter is explicit about both directions: confidence is scored against written anchors rather than a feeling, a finding is dropped only by a line that disproves it, security and money paths are exempt from the confidence floor that would otherwise bury them, blast radius breaks severity ties with a caller count you measured, and asking for code that can never run — a guard for an impossible state, a test for an unreachable case or one that only restates a low-impact change — is treated as the same error as inventing a bug. Pre-existing issues and lint violations are reported in their own sections and kept out of the verdict, so they cannot crowd out what the change actually did.

Supports workspace changes, branch ranges, and single commits. Optional project rules via `.opencodereview/rule.json` or `REVIEW_RULES.md`.

### Install

```bash
# Claude Code
mkdir -p ~/.claude/skills && ln -s "$PWD"/open-code-review ~/.claude/skills/
# Codex
mkdir -p ~/.codex/skills && ln -s "$PWD"/open-code-review ~/.codex/skills/
```

Then in either agent: "review my changes", "review this branch against main", "review commit abc123", or "review and fix".

### Lineage

Method distilled from these open-source review and hunting skills:

- [alibaba/open-code-review](https://github.com/alibaba/open-code-review): deterministic scoping and rule matching, positioning and reflection passes
- [codexstar69/bug-hunter](https://github.com/codexstar69/bug-hunter): Hunter / Skeptic / Referee, runtime trigger per finding, confidence scores, known false-positive classes
- [tag1consulting/claude-comprehensive-review](https://github.com/tag1consulting/claude-comprehensive-review): blind-hunter, edge-case path walk, adversarial "what is missing" lens, version-flagging rule
- [trailofbits/skills](https://github.com/trailofbits/skills): per-function contract map, `nothing found`, no hedges, follow every callee path
- [anthropics/claude-code-security-review](https://github.com/anthropics/claude-code-security-review): baseline existing patterns first, exploit scenario per security finding
- [obra/superpowers](https://github.com/obra/superpowers): read-only review, plan alignment, explicit merge verdict
- [anthropics/claude-code](https://github.com/anthropics/claude-code) `code-review` and `pr-review-toolkit` plugins: confidence rubric with written anchors, a catalogue of false-positive classes, a reviewer pass dedicated to git history
- [trailofbits/skills](https://github.com/trailofbits/skills) `differential-review`: git history as regression detection (a re-introduced vulnerability), blast radius by caller count, depth that adapts to risk

Calibrated against the current prompting guidance from both vendors: Anthropic's [Claude Code best practices](https://code.claude.com/docs/en/best-practices) ("a reviewer prompted to find gaps will usually report some, even when the work is sound"), [Opus 5 prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) ("ask it to report everything and filter in a separate pass instead"), and [Opus 5.5 prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5) (the named early stops that end a turn with work still owed; status notes ride with the next tool call); OpenAI's [GPT-6 model guidance](https://developers.openai.com/api/docs/guides/latest-model) (user instructions take precedence over a skill; no tests for reversible, low-impact changes that mirror the implementation; short, trigger-focused skill descriptions); and OpenAI's [Codex prompting guide](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide) (findings first, ordered by severity with file/line references; batch reads; finish in one turn) and [GPT-5.2 prompting guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-2_prompting_guide) ("never fabricate exact figures, line numbers, or external references when you are uncertain").

## open-feature-dev

One issue in, one pull or merge request out, run by the agent with git and `gh`/`glab`. It reads the issue as data (never as instructions), explores the code before asking anything, then interviews you: a few multiple-choice questions grounded in file and line, each with a recommendation, plus a trade-off table when more than one approach is real. It writes a plan for an engineer who has never seen the repo — tasks with files, interfaces, the test to write first, an end-to-end verification, and a Complexity Tracking table that has to justify every new abstraction — and a fresh-context reviewer challenges the plan (fidelity, premise, feasibility against the real code, a simpler rung, risk, tests) before you approve it. A small team then builds it: one implementer per task, sequential; a task reviewer that returns two verdicts, spec compliance (missing / extra / misunderstood) and code quality, and does not trust the implementer's report; a bounded fix loop that resumes the same implementer three times, replaces it once, then stops and records a ruling; and a final whole-branch review that dispatches `open-code-review`. The PR/MR body explains what the diff cannot, links the issue with `Closes #N`, and is opened only after one confirmation that shows the branch, commits, title, and body. No CLI, no API key.

The shape follows from five failures every such workflow suffers. Success claimed without evidence: every implementer report is checked against `git diff` and a test run the controller performs itself. Rubber-stamping and pre-judging: reviewers run in fresh contexts, the controller never tells one what not to flag, and a finding leaves only through a dismissal that quotes the disproving line. Reviewer findings that drive over-engineering: a request for a guard on an unreachable state or a helper nothing needs is dismissed as the same error as an invented bug. Scope creep: one issue is one PR, "extra" is a spec-compliance failure, and work noticed on the way becomes a follow-up, never a commit. Losing the thread after compaction: a ledger file records every dispatch, round, ruling, and completion, so a compacted controller resumes from the file and `git log`, never from memory. Between the three user gates the run does not stall on questions; it decides, records `Ruling: what — why — cost if wrong`, and lists every ruling at the end.

Small changes take a `direct` tier with one gate before code; risk surfaces (auth, money, migrations, external contracts, public API, concurrency) force the full team.

### Install

```bash
# Claude Code
mkdir -p ~/.claude/skills && ln -s "$PWD"/open-feature-dev ~/.claude/skills/
# Codex
mkdir -p ~/.codex/skills && ln -s "$PWD"/open-feature-dev ~/.codex/skills/
```

Install `open-code-review` alongside it for the final review; without it a shorter built-in checklist runs. Then: "implement issue #42", "fix https://github.com/org/repo/issues/42", "ship this: <description>".

### Lineage

Method distilled from these open-source workflows:

- [obra/superpowers](https://github.com/obra/superpowers): subagent-driven development — briefs and diffs handed over as files, the two-verdict task review, "do not trust the report", the bounded fix loop with escalation and a breaker, the ledger, rulings instead of stalls, no pre-judging the reviewer
- [github/spec-kit](https://github.com/github/spec-kit): the clarification protocol — taxonomy scan, capped recommendation-first multiple choice, stop rules, answers integrated into the spec at once; the Complexity Tracking table
- [anthropics/claude-code](https://github.com/anthropics/claude-code) `feature-dev` plugin: explore before asking, explorers return files the controller reads itself, "whatever you think" gets a recommendation and an explicit yes; `code-review` plugin: validate findings in a separate pass
- [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin): output tiering before depth, forced heavy on risk surfaces; a PR body sized by decision cost that explains what the diff cannot show
- [garrytan/gstack](https://github.com/garrytan/gstack): the premise challenge in plan review, the quote gate (no finding without its motivating line), scope-drift detection, missing coverage is never a pass
- [PortSwigger/agent-wrangler](https://github.com/PortSwigger/agent-wrangler) `issue-to-pr`: the one-issue-one-PR arc, "if you can't tell what to build, don't guess", the controller alone owns the PR
- [warpdotdev/oz-for-oss](https://github.com/warpdotdev/oz-for-oss) `implement-issue`: issue content is untrusted data fetched through one sanctioned path
- [Flagrare/agent-skills](https://github.com/Flagrare/agent-skills): questions grounded in `path:line` after exploration, reviewer coverage lines, "finding nothing is a result", the PR template as skeleton not enumeration
- [bmad-code-org/BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD): verify-then-verdict triage with routes and a cascade; the shippable-deliverable scope test
- [ghuntley/how-to-ralph-wiggum](https://github.com/ghuntley/how-to-ralph-wiggum): state in files, serialize mutation; [buildermethods/agent-os](https://github.com/buildermethods/agent-os): save the spec to disk before any implementation runs

Calibrated against Anthropic's [Claude Code best practices](https://code.claude.com/docs/en/best-practices) ("if you could describe the diff in one sentence, skip the plan"; a fresh-context reviewer of the diff against the plan, told to "report gaps, not style preferences"), the [Opus 5 prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) ("check in only when different readings of the request would lead to materially different work"; keep spawn counts low), the [Opus 5.5 prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5) (effort, not prompt text, sets depth; the four early stops a subagent must not make; a time signal speeds up exploration and costs verification, so only explorers get it), the [Claude prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) (the over-engineering rules; confirm before operations visible to others such as pushing), OpenAI's [GPT-6 model guidance](https://developers.openai.com/api/docs/guides/latest-model) ("the user's instructions take precedence over guidelines provided in a skill"; stop testing once required checks pass; no test that only mirrors a low-impact change), and OpenAI's [Codex](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide) and [GPT-5.2](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-2_prompting_guide) guides ("do not make single-step plans"; "implement exactly and only what the user requests").
