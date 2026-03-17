---
name: run
description: "Start an Outside-In TDD session. Usage: /outside-in-tdd:run <feature description> [--design <path>] [--debug]"
---

# Run Outside-In TDD

Parse the user's arguments and invoke the `outside-in-tdd` orchestrator skill with structured inputs.

## Input Parsing

Extract from the arguments:

1. **Feature requirement**: Everything that is not a flag
2. **Design document**: Value after `--design` or `--doc` flag, or "none"
3. **Debug mode**: Present if `--debug` flag is found

## Execution

Load and execute the `outside-in-tdd` skill with the parsed inputs. The orchestrator handles all phases from there.

## Pre-flight Check

Before starting, verify the project has the required config files:

1. `.claude/tdd-registry.md` — if missing, stop and tell the user to create it (point them to the plugin README)
2. `.claude/business-processes.md` — if missing, stop and tell the user to create it

If both exist, proceed with the orchestrator.

## Examples

```
/outside-in-tdd:run Implement a checkbox that toggles todo completion
/outside-in-tdd:run Add user authentication --design docs/auth-design.md
/outside-in-tdd:run Implement dark mode toggle --debug
```
