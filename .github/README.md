# EXACT Coding workflow -- GitHub Copilot harness

The Red-Green-Refactor workflow for **Copilot CLI** and **VS Code agent mode**,
from one tree. Version: see `VERSION`.

## Layout

| Path | What it is |
|------|------------|
| `skills/tdd/SKILL.md` | The orchestrator. Start here: `/tdd` |
| `skills/{test-list,red,green}/SKILL.md` | The three main-context phases |
| `skills/example-mapping/SKILL.md` | Requirements exploration, before TDD |
| `agents/refactor.agent.md` | Per-cycle refactor, isolated context |
| `agents/end-refactor.agent.md` | Final metric-driven pass over `src/`, once |
| `rules/human-in-the-loop.md` | Autonomy Level -- the single stop-behaviour source |
| `rules/subagent-prompts.md` | What to pass each isolated agent |
| `rules/tdd-with-ts-and-vitest.md` | TypeScript and Vitest conventions |

Copilot discovers `skills/` and `agents/` under `.github/` automatically.
`rules/` is not a Copilot mechanism -- those files are read on demand because
the skills and agents point at them by path.

## Starting a session

Say "let's TDD this kata", or invoke `/tdd` directly. Nothing loads
automatically: a session that never mentions TDD gets no Red-Green-Refactor
discipline, which is what you want when you are fixing a typo.

## What differs between CLI and VS Code

Everything is shared except how a subagent is started.

| | Copilot CLI | VS Code agent mode |
|---|---|---|
| Invoke a phase | `/red`, `/green`, `/test-list` | same, via `/` |
| Delegate refactoring | `/agent refactor`, or the model picks it up as a tool | subagent tool (`#runSubagent`), naming the `refactor` agent |
| Tool permissions | `--allow-tool` and CLI config | VS Code approval prompts and terminal auto-approve settings |

The agents declare no `tools:` list on purpose, so they inherit whatever the
surface grants. That keeps one file working in both places; the cost is that
tool restriction is not enforced from the agent definition the way it is in the
OpenCode and pi harnesses.

## Known gaps against the other harnesses

- **No permission allowlist.** `.claude/settings.json` and `opencode.json` pin
  which commands may run unattended. There is no equivalent checked into this
  tree; batch runs need the surface configured by hand.
- **Delegation is inferred, not forced.** Copilot decides whether to load a
  skill or delegate to an agent. The orchestrator states the requirement
  loudly, but if you are measuring the workflow, verify from the transcript
  that the refactor phases actually ran in a subagent.
- **This tree must not coexist with `.claude/`.** Copilot reads
  `.claude/skills/` and `.claude/agents/` as its own and would see every skill
  twice. That is why each harness lives on its own branch.
