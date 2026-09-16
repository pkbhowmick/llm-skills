# llm-skills

Skills for Claude Code and Codex. Each directory is one skill (`<name>/SKILL.md`). No dependencies beyond git.

## open-code-review

Exhaustive, line-level code review run entirely by the agent. Whole files are read, not hunks. Every changed unit gets a contract map (inputs and trust, assumptions and the line that establishes each, callers including indirect ones, callees including library calls, removed code, refactor equivalence), then is checked against eleven bug-hunting dimensions (logic, edges, error paths, concurrency, resources, security, data integrity, contracts, performance, tests, and git history — whether the change undoes a past fix), an absence pass for what the diff should contain but does not, and a strict style lens. A skeptic pass can dismiss a finding only with a cited line, and every dismissal is reported. Then a second cold read, a verdict that follows fixed rules, and a per-unit coverage grid. No CLI, no API key.

The hunt runs without a severity floor and the filtering happens afterwards, in one pass with its own rules, because a reviewer told to be conservative is conservative about real bugs too — and a reviewer told to find problems finds them whether or not they exist. So the filter is explicit about both directions: confidence is scored against written anchors rather than a feeling, a finding is dropped only by a line that disproves it, security and money paths are exempt from the confidence floor that would otherwise bury them, blast radius breaks severity ties with a caller count you measured, and asking for code that can never run — a guard for an impossible state, a test for an unreachable case — is treated as the same error as inventing a bug. Pre-existing issues and lint violations are reported in their own sections and kept out of the verdict, so they cannot crowd out what the change actually did.

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

Calibrated against the current prompting guidance from both vendors: Anthropic's [Claude Code best practices](https://code.claude.com/docs/en/best-practices) ("a reviewer prompted to find gaps will usually report some, even when the work is sound") and [Opus 5 prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) ("ask it to report everything and filter in a separate pass instead"), and OpenAI's [Codex prompting guide](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide) (findings first, ordered by severity with file/line references; batch reads; finish in one turn) and [GPT-5.2 prompting guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-2_prompting_guide) ("never fabricate exact figures, line numbers, or external references when you are uncertain").
