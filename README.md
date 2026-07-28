# EXACT Coding - Exercises

Exercise setup for the **EXACT Coding** workshop: **EX**ample-guided, **A**I-**C**ollaborative & **T**est-driven Development.

## Workshop

Test-Driven, AI-Assisted Development for Maintainable Code.

This hands-on workshop introduces EXACT Coding -- a pragmatic workflow for AI-assisted development focusing on readability, refactorability, and maintainability. Participants work in pairs and groups of three (driver + two navigators) through short modules with extensive exercises.

### Topics

- Test-Driven Development (TDD)
- Example Mapping
- Mob/Ensemble Programming
- AI Tools (Claude Code and Cursor)
- EXACT Coding Workflow

### Trainers

Ferdi Ade & Marco Emrich

## Setup

There are two ways to set up the project: using the **Dev Container** (recommended) or a **local installation**.

### Option A: Dev Container (recommended)

The repo includes a Dev Container configuration with Node.js, Claude Code, and a restrictive firewall pre-installed.

#### Prerequisites

- [Docker](https://www.docker.com/)
- [VS Code](https://code.visualstudio.com/) with the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
- Claude Code API key or Portkey configuration (see below)

#### API Key Configuration

**Option 1: Direct API key** -- set the environment variable on your host before opening the container:

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

**Option 2: Portkey proxy** -- configure in `~/.claude/settings.json` on your host:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.portkey.ai",
    "ANTHROPIC_AUTH_TOKEN": "dummy",
    "ANTHROPIC_CUSTOM_HEADERS": "x-portkey-api-key: <your-key>"
  }
}
```

The container automatically mounts `~/.claude/` from your host, so your settings are available inside the container.

#### Starting the Dev Container

1. Open the project folder in VS Code (**File -> Open Folder**, not as workspace)
2. VS Code will prompt: "Reopen in Container" -- click it
3. Or manually: `Ctrl+Shift+P` -> **"Dev Containers: Reopen in Container"**
4. Wait for the container to build (first time takes a few minutes)

After startup, run `npm install` in the terminal, then verify with the checks below.

### Option B: Local Installation

#### Prerequisites

- Node.js (v20 or higher)
- npm
- Claude Code (`npm install -g @anthropic-ai/claude-code`)

#### Installation

```bash
npm install
```

### Running Tests

```bash
npm test
```

### Watch Mode

```bash
npm run test:watch
```

## Verify Your Setup

Run the following checks to make sure everything is working (both local and Dev Container).

**1. Node.js installed?**

```bash
node --version
# Expected: v20 or higher (e.g. v24.9.0)
```

**2. Claude Code installed and API key configured?**

```bash
claude -p "respond with: setup ok"
# Expected: "setup ok" (or similar short response)
```

If this hangs or returns an authentication error, your API key is not configured correctly. See the [Claude Code docs](https://docs.anthropic.com/en/docs/claude-code) for setup instructions.

**3. Dependencies installed and tests passing?**

```bash
npm install
npm test
```

Expected test output:

```
 ✓ src/example.spec.ts (1 test)

 Test Files  1 passed (1)
      Tests  1 passed (1)
```

(If you have already worked through exercises, you will see more files and tests
than this — what matters is that everything passes.)

If all checks pass, you're ready for the workshop!

## Agent Configuration

This repo ships the EXACT Coding TDD workflow preconfigured for **four coding
agents**. They are equivalent — use whichever you have. Version **2026-07-28**.

| Agent | Directory | Start a TDD session with |
|-------|-----------|--------------------------|
| Claude Code | `.claude/` | Ask for TDD in plain language ("let's TDD this kata") |
| pi | `.pi/` | `/skill:tdd`, or ask for TDD in plain language |
| OpenCode | `.opencode/` | The `tdd` command |
| Cursor | `.cursor/` | Ask for TDD in plain language |

**You have to ask for the workflow.** None of the four load it automatically. A
session where you never mention TDD gets no Red-Green-Refactor discipline, no
prediction blocks, no checkpoints — which is what you want when you are just
fixing a typo. Say "using TDD" and the whole workflow comes in.

### The workflow

Five phases: Test-List once, then Red-Green-Refactor per test, then a single
End-Refactor pass over everything at the end.

| Phase | Runs in | What happens |
|-------|---------|--------------|
| Test List | Main context | Turns the spec into `it.todo()` entries, ordered simple to complex |
| Red | Main context | Activates ONE test, predicts the failure (the Guessing Game), verifies it fails for the right reason |
| Green | Main context | Writes the minimal code to pass — hardcoded returns are fine |
| Refactor | **Isolated subagent** | Four Rules of Simple Design, naming first, APP mass before/after |
| End-Refactor | **Isolated subagent** | Runs once at the end over the whole `src/`, measuring ESLint smells, cognitive complexity, APP mass and McCabe complexity around each change |

Test-List, Red and Green share one context on purpose — the predictions, error
messages, and minimal implementations only make sense together. Both refactor
phases run in a **fresh, isolated context** on purpose: the refactorer sees the
code as it stands, not the history of how it got there, which is what lets it
judge the result on its own merits.

The End-Refactor pass exists because a per-cycle refactor only ever sees one
file mid-flight. After the last test passes, the design has stabilised and
cross-file duplication and complexity hot spots become visible for the first
time.

### Human-in-the-Loop

The default is **`full-hitl`**: the agent stops and waits for your approval
after Test-List, after Red, and after Refactor — and immediately whenever a
prediction turns out wrong. It does **not** stop after Green; Green is the most
mechanical phase, and stopping there mostly produces "yes, continue".

A wrong prediction is always a hard stop. It means the model's picture of the
system disagrees with reality, and continuing usually compounds the
misunderstanding.

Other levels: `refactor-only`, `red-only`, `every-n-tests N`, `task-end`,
`autonomous`. Change one line at the top of your agent's HITL file:

| Agent | File |
|-------|------|
| Claude Code | `.claude/rules/human-in-the-loop.md` |
| pi | `.pi/rules/human-in-the-loop.md` |
| OpenCode | `.opencode/rules/human-in-the-loop.md` |
| Cursor | `.cursor/rules/human-in-the-loop.mdc` |

That file is the single source of truth. The phase files point at it but contain
no stop logic themselves, so changing the level changes the whole workflow.

### Example Mapping

Separate from TDD, for exploring a feature *before* you write any tests:
`/example-mapping` in Claude Code, or the `example-mapping` command in OpenCode.
It facilitates a session over four card colours — story, rules, examples,
questions — and writes the result to a markdown file. It asks you for the rules
and examples; it does not invent them.

### pi: one extra step

pi has no built-in subagent mechanism, so the refactor phases rely on a small
extension bundled at `.pi/extensions/subagent/`. **On first use pi asks whether
you trust the project — say yes.** If you decline, pi has no way to delegate and
you end up with a workflow that silently skips refactoring.

The other three agents have subagents natively and need nothing extra.

### Where the workflow comes from

The configuration is generated from the workflow research in
`agentic_coding_lab_project` and validated against Claude Opus 4.8. Each
directory carries its own `README.md` and `VERSION`. For the lineage and the
experiments behind it, see `research/workflow-dev/workflow-construction.md` in
that repo.
