# llm-skills

Skills for Claude Code and Codex. Each directory is one skill (`<name>/SKILL.md`). No dependencies beyond git.

## open-code-review

Exhaustive, line-level code review run entirely by the agent. Whole files are read, not hunks. Every changed unit gets a contract map (inputs and trust, assumptions and the line that establishes each, callers including indirect ones, callees including library calls, removed code, refactor equivalence), then is checked against ten bug-hunting dimensions (logic, edges, error paths, concurrency, resources, security, data integrity, contracts, performance, tests), an absence pass for what the diff should contain but does not, and a strict style lens. A skeptic pass can dismiss a finding only with a cited line, and every dismissal is reported. Then a second cold read, a verdict that follows fixed rules, and a per-unit coverage grid. No CLI, no API key.

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
