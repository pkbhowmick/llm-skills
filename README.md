# llm-skills

Skills for Claude Code and Codex. Each directory is one skill (`<name>/SKILL.md`). No dependencies beyond git.

## open-code-review

Exhaustive, line-level code review run entirely by the agent. Every changed unit is checked against ten bug-hunting dimensions (logic, edges, error paths, concurrency, resources, security, data integrity, contracts, performance, tests) plus a strict style lens. Everything found is reported, down to nits, with a per-file coverage grid. Then a skeptic pass that can only dismiss a finding with a cited line, a second cold read, and a per-file coverage grid. No CLI, no API key.

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
