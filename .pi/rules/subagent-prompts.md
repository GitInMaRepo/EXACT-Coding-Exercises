# Subagent Prompt Contracts

The refactor and end-refactor subagents run in **isolated contexts**. They
have no memory of the test-list, red, or green phases. Everything they need
must be in the prompt.

This file defines what to pass. It is workflow methodology, not lab
infrastructure — it stays when the workflow is exported.

## Workflow Sequence

1. **Test List Phase** → Read `.pi/skills/test-list/SKILL.md` (main context)
2. **For each test:**
   - **Red Phase** → Read `.pi/skills/red/SKILL.md` (main context)
   - **Green Phase** → Read `.pi/skills/green/SKILL.md` (main context)
   - **Refactor Phase** → Launch the `refactor` subagent via the `subagent` tool with `agent: "refactor"`, `agentScope: "both"` (isolated context)
3. **Continue** until all tests are implemented and passing
4. **End-Refactor Phase** → Launch the `end-refactor` subagent ONCE via the `subagent` tool with `agent: "end-refactor"`, `agentScope: "both"` (isolated context), over the whole `src/`

## Required Prompt Context for the Refactor Subagent (per cycle)

```
Test file: [path]
Implementation file: [path]
Passing tests: [count]
Recent changes: [one-line summary of the Green phase]
```

After the subagent returns, read its summary and proceed directly to the
next Red phase.

## Required Prompt Context for the End-Refactor Subagent (once, after the last green cycle)

The end-refactor subagent refactors the **whole production tree**. Pass:

```
Implementation files: src/<all non-spec *.ts>
Test files: src/<*.spec.ts>
Passing tests: [count]

Run the final metric-driven refactoring pass over the whole src/.
Iterate ONE change at a time with pre/post measurement (ESLint, cognitive,
APP, McCabe). Stop when no metric improves further or no improvement is
possible.
```

Launch the end-refactor subagent exactly once, after the last per-cycle
refactor returns. After it returns, read its summary.
