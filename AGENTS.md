# ATM Agents

The ATM project includes two custom agents available for development workflows.

## Available Agents

### 1. Dice Roller

**File:** [`.github/agents/dice-roller.agent.md`](.github/agents/dice-roller.agent.md)

**Usage:** Type `@dice-roller` in the chat

**Responsibilities:**

- Rolling dice with true randomness
- Supports various dice types: d6, d20, 3d6, etc.

**Usage Example:**

```bash
@dice-roller roll d20 for an attack
```

---

### 2. Static Analysis Fixer

**File:** [`.github/agents/static-fixer.agent.md`](.github/agents/static-fixer.agent.md)

**Usage:** Type `@static-fixer` in the chat

**Responsibilities:**

- Automatically fix C/C++ static analysis violations
- Prioritize fixes by severity
- Verify that fixes don't break the code
- **Always** rolls dice at the end (automatically!)

**Workflow:**

1. Analyzes the entire codebase
2. Groups violations by file and severity
3. Fixes issues, starting with the highest severity
4. Builds the project to verify fixes
5. Invokes `@dice-roller` at the end to celebrate success!

**Usage Example:**

```bash
@static-fixer fix all static analysis violations in the ATM project
```

---

### 3. Coverage Booster

**File:** [`.github/agents/coverage-booster.agent.md`](.github/agents/coverage-booster.agent.md)

**Usage:** Type `@coverage-booster` in the chat

**Responsibilities:**

- Measuring current C/C++ test coverage using `cpptestct`
- Identifying uncovered lines, branches, and functions
- Writing new unit tests to close coverage gaps
- Verifying improvement by re-running coverage

**Workflow:**

1. Runs `cpptestct` to capture baseline coverage
2. Reads source files to understand uncovered code
3. Writes new test cases targeting the gaps
4. Builds the project to verify tests compile
5. Re-runs `cpptestct` to confirm coverage improved

**Usage Example:**

```bash
@coverage-booster check coverage and add tests
```

---

## How It Works

Agents utilize the following skills:

| Agent                | Skill                 | Source        |
|----                  |-----                  |-------        |
| Dice Roller          | `roll-dice`           | Built-in skill|
| Static Analysis Fixer| `fix-static-analysis` | Built-in skill|
| Coverage Booster     | *(inline workflow)*   | —             |

### Workflow: Static Analysis Fixing

```text
┌──────────────────────────────────────┐
│   @static-fixer                      │
│   "fix static analysis violations"   │
└────────────────┬─────────────────────┘
                 │
                 ├─► Run fix-static-analysis skill
                 │   ├─ Full codebase analysis
                 │   ├─ Group violations by severity
                 │   ├─ Fix issues (highest severity first)
                 │   └─ Verify build process
                 │
                 └─► On success: Run @dice-roller
                     └─ Roll dice to celebrate!
```

---

## Tips

1. **Dice Roller** is simple and independent — use it on its own
2. **Static Fixer** will always invoke Dice Roller at the end — you don't need to do it manually
3. **Coverage Booster** runs a full measure → analyze → write → verify loop automatically
4. All agents have access to only the minimal tools needed for their role
5. Agents automatically use appropriate skills without user intervention

---

## Adding New Agents

To add a new agent:

1. Create a `.agent.md` file in `.github/agents/`
2. Add YAML frontmatter with `description` and `tools`
3. Write clear instructions for the agent
4. Update this README file

Example frontmatter:

```yaml
---
description: "Use when you need to [specific task]"
name: "Agent Name"
tools: []
user-invocable: true
---
```
