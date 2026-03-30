# GitHub Copilot Instructions — ATM Project

## Agent Usage Rules

### Dice Rolling
When asked to roll dice (d6, d20, 3d6, etc.) or generate any random dice result,
**always** delegate to the dedicated **Dice Roller** agent via `runSubagent` with
`agentName: "Dice Roller"`. Never roll dice inline (e.g., via terminal or code snippet).

### Static Analysis Fixing
When asked to fix static analysis violations, use the **Static Analysis Fixer** agent
(`@static-fixer`). It automatically invokes the Dice Roller agent upon success.

### Test Coverage
When asked to check test coverage, measure coverage, or add tests to improve coverage,
**always** delegate to the **Coverage Booster** agent (`@coverage-booster`).
Never run `cpptestct` inline or write tests outside of this agent.

See [AGENTS.md](../AGENTS.md) for full agent documentation.
