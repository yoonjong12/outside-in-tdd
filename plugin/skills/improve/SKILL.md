---
name: improve
description: "Analyze and improve the TDD workflow configuration. Usage: /outside-in-tdd:improve [target]. Targets: skills, agents, registry, all."
---

# Improve Outside-In TDD

Analyze the current project's TDD configuration and suggest or apply improvements based on past friction.

## Input Parsing

Extract from the arguments:

- **Target** (optional, default "all"): `skills` | `agents` | `registry` | `all`

## Process

### Step 1: Collect evidence

Gather signals about what's not working:

1. **Debug reports**: Search for recent debug report output in conversation history or git log messages mentioning debug issues
2. **Registry gaps**: Read `.claude/tdd-registry.md` and check for:
   - Missing failure routing entries
   - Missing file path → layer mappings
   - Commands that reference tools not in package.json
3. **Skill coverage**: For each layer in the registry, verify a testing skill exists at the expected path
4. **Agent friction**: Check if any agent contracts reference fields that the orchestrator doesn't provide

### Step 2: Present findings

Output a structured report:

```
## TDD Workflow Health Check

### Issues Found
- [SEVERITY] [component]: description

### Suggested Improvements
- [component]: what to change and why

### No Issues
- [component]: looks good
```

### Step 3: Apply (with user approval)

For each suggested improvement, ask the user before making changes. Group related changes together. Load the `self-improvement` skill for guidance on where changes belong:

| Change type | Where it goes |
|-------------|---------------|
| Workflow logic | Agents or orchestrator (plugin-level — warn user this modifies the plugin) |
| Test patterns | Testing skills (project-level) |
| Code patterns | Implementation skills (project-level) |
| Layer config | Registry (project-level) |
| Domain knowledge | business-processes.md (project-level) |

## Examples

```
/outside-in-tdd:improve                  # Check everything
/outside-in-tdd:improve skills           # Check testing/implementation skills only
/outside-in-tdd:improve registry         # Check tdd-registry.md only
```
