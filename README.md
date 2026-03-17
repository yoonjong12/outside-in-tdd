# Outside-In TDD

A Claude Code plugin that runs Outside-In Test-Driven Development. Starts from E2E tests, drills down through architecture layers, writing tests before implementation at every level.

## What It Does

When you ask Claude Code to implement a feature, the plugin orchestrates four phases:

```
Phase 1: PLAN          → Explore codebase, design architecture, determine test strategy
                         ⏸️ User reviews and approves the plan

Phase 2: E2E TEST      → Write a failing end-to-end test that defines "done"
                         ⏸️ User reviews the assertion list before test is written

Phase 3: INNER CYCLES  → One layer at a time (e.g., backend → frontend)
  ├ RED                  ⏸️ User reviews the behaviour list before tests are written
  ├ GREEN               → Implement minimal code to pass tests
  ├ REFACTOR            → Improve code quality while keeping tests green
  └ (repeat until E2E passes)

Phase 4: COMPLETION    → Full verification + mutation testing gate
```

The ⏸️ checkpoints are where the plugin pauses for your input. You control **what** gets tested; the plugin handles **how**.

## Install

```bash
# Add this repo as a marketplace
claude plugin marketplace add yoonjong12/outside-in-tdd

# Install at user level (available in all projects)
claude plugin install outside-in-tdd@yoonjong12-outside-in-tdd --scope user
```

Or for a single session:

```bash
claude --plugin-dir /path/to/outside-in-tdd-plugin
```

## Project Setup

The plugin provides the workflow engine. Each project provides its own configuration in `.claude/`:

```
your-project/
└── .claude/
    ├── tdd-registry.md              # REQUIRED — layers, commands, failure routing
    ├── business-processes.md         # REQUIRED — domain workflows and personas
    ├── infrastructure.md             # REQUIRED — Docker/DB debugging commands
    └── skills/
        ├── <your-e2e-testing>/skill.md        # E2E test patterns
        ├── <your-frontend-testing>/skill.md   # Frontend test patterns
        ├── <your-backend-testing>/skill.md    # Backend test patterns
        ├── <your-frontend-impl>/skill.md      # Frontend code conventions (optional)
        └── <your-backend-impl>/skill.md       # Backend code conventions (optional)
```

### tdd-registry.md

Maps architecture layers to skills and defines project commands:

```markdown
## Layers

| Layer    | Testing Skill          | Implementation Skill    | Description              |
|----------|------------------------|-------------------------|--------------------------|
| e2e      | cypress-e2e-testing    | —                       | Cross-layer E2E          |
| frontend | vitest-frontend        | react-implementation    | Components, hooks, pages |
| backend  | vitest-backend         | express-implementation  | API routes, lib utilities|

## Commands

| Purpose           | Command                         |
|-------------------|---------------------------------|
| Specific test     | `npx vitest run <path>`         |
| Full verification | `npm test`                      |
| Mutation testing  | `npx stryker run --since main`  |
```

See the [starter repo](https://github.com/yoonjong12/outside-in-tdd-starter) for a complete example with Jest, Cypress, and React.

## Usage

Open Claude Code in your project and describe a feature:

```
Implement a checkbox on each todo item that toggles completion status.
```

The `outside-in-tdd` skill triggers automatically for feature requests (keywords: "implement", "add feature", "build", "create").

### With an existing design document

```
Implement user authentication. Design doc: docs/auth-design.md
```

The planner will use your design document as a starting point instead of deriving everything from scratch.

### Debug mode

```
Implement a dashboard page --debug
```

Every agent reports instruction issues and execution friction. A consolidated debug report is output at the end — use it to tune your skills and registry.

## User Checkpoints

### Plan Review (Phase 1)

After the planner drafts the implementation plan, you review it before any code is written:

```
## Implementation Plan
Domain: Todo Management / step 3
Architecture: frontend(TodoItem modify) + backend(PATCH route new)
Test Strategy: E2E (multi-layer)
Approach: ...

Review the plan above. Approve or request changes.
```

### E2E Assertion List (Phase 2)

Before the E2E test is written, you confirm what it will verify:

```
E2E assertion list:
1. Click checkbox → strikethrough appears
2. Click again → strikethrough removed
3. Page refresh → state persists

Add, modify, or remove items?
```

### Layer Behaviour List (Phase 3)

Before each layer's unit tests are written, you confirm the behaviours in Given/When/Then format:

```
backend layer test behaviour list:

src/pages/api/todos/[id].ts:
- Given: todo exists with completed=false / When: PATCH /api/todos/123 / Then: completed=true
- Given: no todo with id 999 / When: PATCH /api/todos/999 / Then: 404
- Given: any state / When: GET /api/todos/123 / Then: 405

Mock boundaries:
- Database: mock PrismaClient — producer contract: returns {id, title, completed, createdAt}

Add, modify, or remove items?
```

## Commands

| Command | Purpose |
|---------|---------|
| `/outside-in-tdd:run <feature> [--design <path>] [--debug]` | Start a TDD session. Pre-flight checks project config, then invokes the orchestrator. |
| `/outside-in-tdd:improve [skills\|agents\|registry\|all]` | Health check — analyzes registry, skills, and agents for gaps and friction, applies fixes with user approval. |

## Plugin Components

| Component | Purpose |
|-----------|---------|
| `skills/outside-in-tdd/` | Orchestrator — sequences phases, routes failures, manages the RED-GREEN-REFACTOR cycle |
| `skills/self-improvement/` | Meta-skill — helps Claude update the workflow itself |
| `agents/tdd-planner.md` | Explores codebase, produces implementation plan |
| `agents/tdd-test-writer.md` | Writes failing tests for a layer |
| `agents/tdd-implementer.md` | Implements minimal code to pass tests |
| `agents/tdd-refactorer.md` | Improves code quality while keeping tests green |
| `agents/test-backfiller.md` | Kills surviving mutants from mutation testing |
| `agents/*-contract.md` | Data shape contracts between agents |
| `agents/debug-protocol.md` | Debug reporting format |
