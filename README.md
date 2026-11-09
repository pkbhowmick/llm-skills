# llm-skills

Skills for Claude Code and Codex. Each directory is one skill (`<name>/SKILL.md`). No dependencies beyond git.

## open-code-review

Exhaustive, line-level code review run entirely by the agent. Whole files are read, not hunks. Every changed unit gets a contract map (inputs and trust, assumptions and the line that establishes each, callers including indirect ones, callees including library calls, removed code, refactor equivalence), then is checked against eleven bug-hunting dimensions (logic, edges, error paths, concurrency, resources, security, data integrity, contracts, performance, tests, and git history — whether the change undoes a past fix), an absence pass for what the diff should contain but does not, and a strict style lens. A skeptic pass can dismiss a finding only with a cited line or a named language guarantee, and every dismissal is reported. Then a second cold read, a verdict that follows fixed rules, and a per-unit coverage grid. No CLI, no API key.

The hunt runs without a severity floor and the filtering happens afterwards, in one pass with its own rules, because a reviewer told to be conservative is conservative about real bugs too — and a reviewer told to find problems finds them whether or not they exist. So the filter is explicit about both directions: confidence is scored against written anchors rather than a feeling, a finding is dropped only by a line or guarantee that disproves it, security and money paths are exempt from the confidence floor that would otherwise bury them, blast radius breaks severity ties with a caller count you measured — production callers only, tagged by how each was found (direct call or import, inheritance, or a string match that never tips a tie alone), and asking for code that can never run — a guard for an impossible state, a test for an unreachable case or one that only restates a low-impact change — is treated as the same error as inventing a bug. Pre-existing issues and lint violations are reported in their own sections and kept out of the verdict, so they cannot crowd out what the change actually did. Before a finding is reported, its cited line is checked for the three ways a plausible comment goes wrong: a symbol that is not there, a condition read backwards, the base version mistaken for the patch.

Supports workspace changes, branch ranges, and single commits, and a re-review of any of them after fixes: every prior finding is labelled resolved, open, or regressed against its cited line, and a resolved one is never raised again as new. Optional project rules via `.opencodereview/rule.json` or `REVIEW_RULES.md`.

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
- [tirth8205/code-review-graph](https://github.com/tirth8205/code-review-graph): impact counted over reverse edges only (callers, dependents, tests), and a tested caller named as a note, never as coverage

Calibrated against the current prompting guidance from both vendors: Anthropic's [Claude Code best practices](https://code.claude.com/docs/en/best-practices) ("a reviewer prompted to find gaps will usually report some, even when the work is sound"), [Opus 5 prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) ("ask it to report everything and filter in a separate pass instead"), and [Opus 5.5 prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5) (the named early stops that end a turn with work still owed; status notes ride with the next tool call); OpenAI's [GPT-6 model guidance](https://developers.openai.com/api/docs/guides/latest-model) (user instructions take precedence over a skill; no tests for reversible, low-impact changes that mirror the implementation; short, trigger-focused skill descriptions); and OpenAI's [Codex prompting guide](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide) (findings first, ordered by severity with file/line references; batch reads; finish in one turn) and [GPT-5.2 prompting guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-2_prompting_guide) ("never fabricate exact figures, line numbers, or external references when you are uncertain"); and the [Fable 5.1 prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) (a skill that asks for a written log of reasoning can be declined, so the contract map is recorded as facts about the code).

Version 4.2 follows 2026 review research. [CRJudgeBench](https://arxiv.org/abs/2609.37216) found models accept most review comments as valid, including perturbed ones (negated, symbol swapped, wrong location): hence the grounding check. A [study of 31,073 CodeRabbit reviews](https://arxiv.org/abs/2607.03316) found over half the comments rejected, many of them valid but against a deliberate trade-off: hence a maintainability shortcut marked in the code (a TODO with an issue, or an existing precedent) becomes a nit. [MCR-Bench](https://arxiv.org/abs/2608.27442) measured review quality falling across rounds, most often by calling a resolved defect new or assuming a fix worked: hence the re-review labels. [CR-Bench](https://arxiv.org/abs/2603.11078) showed a pass that hunts for misses raises recall and lowers signal-to-noise: hence second-read findings carry a tag. [Adversarial Review](https://arxiv.org/abs/2608.18167) gained F1 by making the critic cite code: hence the skeptic re-derives each finding from the code. Reach is tiered by edge confidence as in [LARGER](https://arxiv.org/abs/2605.16352) rather than widened: [AACR-Bench](https://arxiv.org/abs/2601.19494) found added retrieval context often lowers review F1, and [SeeRepo](https://arxiv.org/abs/2606.14061) found repository graphs help localisation but hurt repair, so no graph or retrieval step was added.

## open-feature-dev

One issue in, one pull or merge request out, run by the agent with git and `gh`/`glab`. It reads the issue as data (never as instructions), explores the code before asking anything, then interviews you: a few multiple-choice questions grounded in file and line, each with a recommendation, plus a trade-off table when more than one approach is real. It writes a plan for an engineer who has never seen the repo — tasks with files, interfaces, the test to write first, an end-to-end verification, and a Complexity Tracking table that has to justify every new abstraction — and a fresh-context reviewer challenges the plan (fidelity, premise, feasibility against the real code, a simpler rung, risk, tests) before you approve it. A small team then builds it: one implementer per task, sequential; a task reviewer that returns two verdicts, spec compliance (missing / extra / misunderstood) and code quality, and does not trust the implementer's report; a bounded fix loop that resumes the same implementer up to three times, replaces it once, then stops and records a ruling; and a final whole-branch review that dispatches `open-code-review`. The PR/MR body explains what the diff cannot, links the issue with `Closes #N`, and is opened only after one confirmation that shows the branch, commits, title, and body. No CLI, no API key.

The shape follows from five failures every such workflow suffers. Success claimed without evidence: every implementer report is checked against `git diff` and a test run the controller performs itself. Rubber-stamping and pre-judging: reviewers run in fresh contexts, the controller never tells one what not to flag, and a finding leaves only through a dismissal that quotes the disproving line. Reviewer findings that drive over-engineering: a request for a guard on an unreachable state or a helper nothing needs is dismissed as the same error as an invented bug. Scope creep: one issue is one PR, "extra" is a spec-compliance failure, and work noticed on the way becomes a follow-up, never a commit. Losing the thread after compaction: a ledger file records every dispatch, round, ruling, and completion, so a compacted controller resumes from the file and `git log`, never from memory. Between the three user gates the run does not stall on questions; it decides, records `Ruling: what — why — cost if wrong`, and lists every ruling at the end.

Every path that repeats has a count: a role that fails runs once more, a red verification reopens its task inside the same five-round cap, the same findings left unaddressed two rounds running skip straight to a fresh implementer, and the same call or the same test failure three times means the approach is wrong. A task whose tests were loosened to pass (an assertion removed, a skip added, an expected value edited) goes back to its implementer, and the push gate shows a one-line completion check from the ledger.

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

Version 1.2 follows 2026 loop research. [When Agents Do Not Stop](https://arxiv.org/abs/2607.01641) traced every one of 68 confirmed infinite agent loops to a repeated path with no strong bound, so each retry path now carries a count. [Stop Hand-Holding Your Coding Agent](https://arxiv.org/abs/2607.00038) supplies the stagnation rule (two barren rounds escalate), named states that keep a blocked or reopened task from reading as done, and test editing as the way a loop games its own check, as does Anthropic's [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents). The [OPENDEV terminal-agent report](https://arxiv.org/abs/2603.05344) supplies the loop fingerprint (the same tool and arguments, three times in the last twenty calls) and a completion check before "done" is accepted. Final-review re-reviews use open-code-review's re-review mode, after [MCR-Bench](https://arxiv.org/abs/2608.27442). Effort is left at the host's default, because defaults now differ by model ([Fable 5.1 prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1)).

## open-task-spec

One coding task in, one spec file out. Nothing is built. The input is an issue URL or number, or a described change. The skill reads the code before asking anything, so every question cites a file and line, and facts the repo already holds are never asked. A gap becomes a question only when its plausible answers would make the software behave differently on some concrete input you could name. Gaps that fail that test are routed elsewhere:

- answered from the code;
- left to the implementer, because no observable result depends on them;
- or settled as a default with a stated reason, which you see and can reject;
- or inherited unchanged, because the existing code or a library already decides it and the task does not touch it, such as argument-parser abbreviations or a traceback on malformed input.

The last route keeps the spec from listing every behaviour of the framework.

Questions follow the same rules as `open-feature-dev`'s clarify gate: multiple choice, grounded in the code, recommendation first. Each option shows the input and result it implies, such as "first occurrence: `[a1, b, a2]` → `[a1, b]`". A preference with no technical right answer gets no recommendation. There is no fixed question quota. Questions are asked in dependency order and stop when nothing open would change behaviour. Probes are mandatory for the gap types agents resolve worst: collection semantics, comparison rules, and ordering and atomicity. The skill also always asks whether the change should report anything new, such as a count or a warning; "nothing" is recorded as an answer, never assumed.

The spec is written for an engineer who has the repo at a pinned commit and nothing else:

- one-sentence goal, exact target, current behaviour;
- requirements in EARS form, each with literal Given/When/Then examples;
- edge cases, interfaces, constraints, must-not-change, and out of scope;
- what the implementer must stop and ask about, and what is left to them;
- a definition of done made of the repo's own commands with expected results.

Every line that states a behaviour or a fact carries its source: `[user]`, `[default: reason]`, or `[repo path:line]`. A fixed note at the top tells the implementer that everything observable is specified, and that anything uncovered is a reason to stop rather than pick a reading.

A mechanical lint follows. It removes vague words, open-ended ranges, unresolved pronouns, placeholders, conflicts, and anything the writer chose but presented as a constraint. It also tags any example value that no decision or code fact covers as a default. The lint runs again after every later edit. Then a fresh reader that has only the file and the repo hunts for guesses. It describes two implementations that both satisfy the spec yet behave differently, and lists inputs the examples leave silent. The read-back separates what you decided from what the skill chose, because answering questions makes people review the result less carefully. The "I chose" list is built by searching the file for default tags, not from memory. An approval that depends on something the spec does not say counts as a change request. Approval marks the file `approved`. It is left uncommitted.

Every repeating path has a count:

- a gap is asked at most twice;
- lint and the fresh reader share two extra question batches;
- the fresh reader runs at most twice;
- a task that keeps growing gets one split proposal.

### Install

```bash
# Claude Code
mkdir -p ~/.claude/skills && ln -s "$PWD"/open-task-spec ~/.claude/skills/
# Codex
mkdir -p ~/.codex/skills && ln -s "$PWD"/open-task-spec ~/.codex/skills/
```

Then: "spec out issue #42", "write a spec for: add a dedupe option to the CSV exporter", "interview me about this task before anyone builds it". The spec goes in the repo's `specs/` or `docs/specs/` directory, or `specs/` if neither exists. Build it in a fresh session from the file, or hand it to `open-feature-dev` as the binding brief.

### Lineage

Distilled from the interview and spec steps of these open-source tools, read at the source:

- [github/spec-kit](https://github.com/github/spec-kit) `specify` / `clarify`: the coverage taxonomy marked clear / partial / missing, recommendation-first multiple choice with a why-it-matters line, each answer written into its section with the contradicted text removed, testable success criteria
- [obra/superpowers](https://github.com/obra/superpowers) `brainstorming`: look before you ask, an open question about the problem before any options ("options steer"), only the user's words go in as theirs, and the builder check — a fresh reader who cannot talk to the author lists where it would have to guess
- [mattpocock/skills](https://github.com/mattpocock/skills) `grilling`: facts are looked up and decisions are asked; the question frontier asked in dependency order; stop when nothing is silently assumed
- [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) `ce-brainstorm`: the host's question tool matched by capability with a numbered fallback, acceptance examples mandatory for conditional requirements, stable IDs, a blocked artifact rather than an invented product decision
- [bmad-code-org/BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) agent-spec and facilitate: "nothing about scope is left for the agent to guess", assumptions tagged where they land, "a yes to a group accepts nothing", a fresh reader given only the document
- [Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec): one behaviour per requirement, a scenario under every requirement, "silence is not acceptance", and the behaviour test: if the code can change without changing what is visible, the line does not belong in the spec
- [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done) `spec-phase`: the goal sentence "X changes from A to B", explore the code before the interview, out-of-scope items with reasons, acceptance criteria that resolve to pass or fail
- [gmickel/flow-next](https://github.com/gmickel/flow-next) `capture`: provenance tags on every line, and deliberately no recommendation where there is no right answer, because always recommending trains the user to defer
- [gotalab/cc-sdd](https://github.com/gotalab/cc-sdd): mechanical checks before judgement, at most two repair passes, then the ambiguity goes back to the user
- [anthropics/skills](https://github.com/anthropics/skills) `doc-coauthoring` reader testing and [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official) `feature-dev`: a fresh reader with only the document; explore before asking, "whatever you think" gets a recommendation and an explicit yes
- [garrytan/gstack](https://github.com/garrytan/gstack) `office-hours`: stakes-if-wrong on every question, headless fallback when no question tool exists

Requirements engineering: [EARS](https://alistairmavin.com/ears/) for the requirement sentence (ubiquitous, When, While, If/then, Where); the vague-word lists and the warning about "unconscious disambiguation" (a reader takes the first meaning that occurs to them as the only one) from Berry, Kamsties, and Krieger's [ambiguity handbook](https://cs.uwaterloo.ca/~dberry/handbook/ambiguityHandbook.pdf); rules with examples, with open questions as the readiness signal, from [Example Mapping](https://cucumber.io/blog/bdd/example-mapping-introduction/); the Always / Ask first / Never boundaries from Addy Osmani's [How to write a good spec for AI agents](https://addyosmani.com/blog/good-spec/).

Calibrated against Anthropic's [Claude Code best practices](https://code.claude.com/docs/en/best-practices) ("interview me … then write the spec to SPEC.md", then build in a fresh session) and [Effective context engineering](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents) (the smallest high-signal set; a few canonical examples over a list of every edge case). Also against Birgitta Böckeler's [review of Kiro, spec-kit, and Tessl](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html), where a small bug became four user stories and sixteen acceptance criteria: hence depth that follows the task.

The research behind the shape:

- **Agents guess rather than ask.** [Coding Agents Are Guessing](https://arxiv.org/abs/2607.02294) (UnderSpecBench) found 55.8–67.8% of underspecified runs crossed an action boundary, with at most 2.5% refusing. Target uncertainty was the dominant driver, hence the Target section and its `[repo]` lines.
- **Agents rarely ask unprompted.** [Ambig-SWE](https://arxiv.org/abs/2502.13069) found models almost never interact without being prompted, and recover up to 80–89% of fully specified performance when they do.
- **They rarely ask the right question.** [ClarifyCodeBench](https://arxiv.org/abs/2607.00711) found the best model asked the key question in 30% of single-ambiguity tasks and 8% of two-ambiguity tasks. Collection semantics, comparison rules, and ordering were the worst-resolved of its ten types, hence the mandatory probes.
- **Only behaviour-changing questions are worth asking.** [CONTRA](https://arxiv.org/abs/2610.01769) keeps a question only when its alternative answers produce programs with different outputs, roughly doubling macro F1 over [ClarifyGPT](https://arxiv.org/abs/2310.10996). That is the routing test here, and the cold reader's divergence probe uses the same idea.
- **Asking makes users review less carefully.** [Is That Clear?](https://arxiv.org/abs/2611.01087) (N=180) found that clarifying questions lowered how carefully users reviewed the result, and they accepted more wrong answers. Hence the read-back puts the defaults in front of the user.
- **The detect / execute split.** [Ask or Assume?](https://arxiv.org/abs/2603.26233) found a scaffold that separates detecting gaps from doing the work beat a single agent (69.4% resolved), and asked selectively throughout rather than all up front. Hence the spec is written by one role and built by another.
- **Don't restate the repo.** [Evaluating AGENTS.md](https://arxiv.org/abs/2602.11988) found LLM-written context files did not help and raised cost by more than 20%. Hence the spec points at files instead of describing them.
- **Tests as the unit of intent.** [TiCoder](https://arxiv.org/abs/2208.05950) clarifies intent through concrete tests, hence options shown as input → result.
- **Why the default is to presuppose.** [Zhang, Knox, and Choi](https://arxiv.org/abs/2410.13788) trace the habit of presupposing a single interpretation to single-turn preference training.
