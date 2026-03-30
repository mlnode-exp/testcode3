---
name: fix-static-analysis
description: Automatically fix C/C++test static analysis violations across the project by running analysis, prioritizing issues by severity, and applying safe corrections. Use when the user asks to fix static analysis issues, clean C/C++test violations, or improve compliance in CI/CD. Do not use for manual review-only requests or non-C/C++ projects.
---

# Fix Static Analysis Violations Skill

Automates the complete workflow for identifying and fixing C/C++test static analysis violations across the entire project, with priority given to highest-severity issues.

---

## Role & Boundaries

- **Input**: C/C++ project with existing C/C++test violations
- **Output**: Fixed source files, build-verified, with full change logs
- **Mode**: Fully automated — no user interaction during execution
- **Scope**: Only modify source files (`.cxx`, `.hxx`, `.cpp`, `.h`). Never modify reports, build config, or project structure.
- **Preserve semantics**: All fixes must preserve original program logic and data flow

---

## Workflow

### Phase 1: Discovery (sequential)
1. Run `run_static_analysis` with empty `source_file` to analyze the full project
   - If fails → retry once with explicit project folder path
   - If retry fails → abort, exit code `2`
2. Run `get_violations_from_report_file` to fetch all violations from `reports/report.xml`
3. Group violations by: **file → severity → rule_id**

### Phase 2: Fix (batch by file)
4. For each file, starting with the file containing highest-severity violations:
   - For each violation's `rule_id`: call `get_rule_documentation` (sequentially)
   - Generate fix based on rule documentation
   - Apply all fixes for the file in one `multi_replace_string_in_file` call
   - Run build (`make` or equivalent)
   - If build fails → revert batch, log failure, continue to next file

### Phase 3: Re-verification
5. Re-run `run_static_analysis` on the full project
6. Compare final report to initial — log what was fixed, what remains

### Phase 4: Report & Exit
7. Output summary (see Logging section), exit with appropriate code

---

## Behavioral Rules

**MCP tool sequencing (CRITICAL):**
- All MCP tools (`run_static_analysis`, `get_violations_from_report_file`, `get_rule_documentation`) MUST execute **sequentially, never in parallel**
- Wait for each tool result before invoking the next
- Reason: prevents race conditions and ensures state consistency across analysis passes

**Batch by file, not by violation:**
- Collect all fixes for a file → apply in one batch → build → move on
- Reason: if File A's batch breaks the build, File B is untouched and diagnosable

**Never ask for user input:**
- Generate and apply all fixes automatically based on rule documentation
- Log everything; user reviews logs post-execution

---

## Decision Logic

| Condition | Action |
|-----------|--------|
| Initial analysis fails | Retry with explicit path → if still fails: exit(2) |
| No violations found | Log success → exit(0) |
| Build fails after batch | Revert batch → log failure → continue to next file |
| Severity 1 violations remain after all fixes | exit(1) |
| All violations fixed + build passes | exit(0) |
| Fix ambiguous or requires refactoring | Skip → log as deferred |

---

## Failure Modes & Mitigation

### When to Revert a Fix
1. **Syntax error introduced**: Pattern doesn't fit this code structure — revert immediately
2. **Build fails**: Review rule documentation for alternative pattern; revert this batch
3. **Logic semantic change**: Fix alters program behavior (e.g., guards code that must always execute) — revert, add comment, skip

### When to Leave a Violation Unfixed (defer)
1. **False positive**: Rule documentation shows the code is actually safe (e.g., pointer guaranteed non-null by caller)
2. **Intentional design**: Code is safe by construction — add explanatory comment and log as deferred
3. **Requires refactoring**: Fix needs restructuring multiple functions — log for manual review, do not attempt

---

## How to Read Rule Documentation

When calling `get_rule_documentation` for a `rule_id`:
- Extract: what the rule detects, why it's dangerous, recommended fix pattern
- Skip generic descriptions; focus on the concrete technical constraint
- If documentation is ambiguous → defer the violation, do not guess

---

## Logging Format

### Per Fix
```
[FIXED] {file}:{line} | Rule {rule_id} (Severity {severity})
  Before: {1-3 lines of original code}
  After:  {1-3 lines of fixed code}
```

### Per Build
```
[BUILD] Batch: {file}
  Result: PASS | FAIL — {error message}
```

### Final Summary
```
[SUMMARY]
  Fixed   (S1/S2/S3): X / Y / Z
  Remaining (S1/S2/S3): A / B / C
  Build Status: PASS | FAIL
  Files Modified: {list}
  Deferred: {count} violations — see log for details
```

## Exit Codes

- `0` — All targeted violations fixed, build passes
- `1` — Severity 1 violations remain, or final build failed
- `2` — Initial analysis failed after retry

---

## Related Tools

- `run_static_analysis` – Execute C/C++test analysis on the project
- `get_violations_from_report_file` – Retrieve violations from `reports/report.xml`
- `get_rule_documentation` – Fetch fix patterns for a specific rule_id
- `read_file` – Review source code context around a violation
- `replace_string_in_file` / `multi_replace_string_in_file` – Apply fixes
- `run_in_terminal` – Build project to verify fixes
- `manage_todo_list` – Track progress through phases

## Additional Resources

- [EXAMPLES.md](references/EXAMPLES.md) – Example prompts to invoke this skill
