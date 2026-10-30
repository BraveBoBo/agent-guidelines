# agent-guidelines

Working rules for AI coding agents, distilled from a robotics research project and written to be project-agnostic. They aim at fewer unnecessary changes, fewer rewrites from over-engineering, root-cause fixes instead of workarounds, and replies a reader can act on.

## Files

| File | For | Use |
|---|---|---|
| `CLAUDE.md` | Claude Code | Copy into a repository root; Claude Code loads it automatically |
| `AGENTS.md` | Codex and other agents that read `AGENTS.md` | Copy into a repository root |

Both carry the same rules with the same section numbers 1–18; `CLAUDE.md` adds a figure-style section (19), so its paper-format section is 20.

## Rules at a glance

| # | Rule | In short |
|---|---|---|
| 1 | Think before coding | State assumptions, surface tradeoffs, ask when unclear |
| 2 | Simplicity first | Minimum code for the task; nothing speculative |
| 3 | Surgical changes | Touch only what the request needs |
| 4 | Goal-driven execution | Turn tasks into verifiable checks and loop until they pass |
| 5 | No fallbacks | Find the root cause and fix it there; no default-when-missing paths or swallowed errors |
| 6 | Reuse existing code | Read the current implementation before adding a second one |
| 7 | Delete unneeded code | Remove dead features outright; no old-data compatibility shims |
| 8 | Naming | Google Python Style Guide names and imports; tunables in YAML config groups |
| 9–10 | Python style and simplicity | Idiomatic Python; validate once at the boundary; no premature helpers, abstractions or `typing.Any`; full type hints on public interfaces |
| 11 | Delegate | The main agent plans and verifies; subagents implement |
| 12 | Simplify and review every change | edits → simplify → code-review → tests → commit |
| 13 | Worktrees and branches | Worktrees under `.claude/worktrees/`; rebase and fast-forward; delete merged branches; never drop unmerged work |
| 14 | Branch names | At most three hyphenated words, no slashes or `feat/`/`fix/` prefixes |
| 15 | Chinese by default | Reply in Chinese unless asked otherwise; code and terms stay in English |
| 16–18 | Replies, paths, documents | Short tables; state inputs, outputs and where they are; full absolute paths, one per line; no history |
| 19–20 (CLAUDE.md) | Figures and papers | TPAMI figure style; IEEE TPAMI paper format |

## Adapting

- Sections 15 (language), 19 (figure style) and 20 (paper format) reflect one team's preferences; drop or edit them for other projects.
- Placeholders such as `<config>/<group>/`, `figures/<intent>/<name>/` and the integration branch name (`dev` in the examples) stand for your project's own layout.
- Explicit instructions from the user always take precedence over these defaults.
