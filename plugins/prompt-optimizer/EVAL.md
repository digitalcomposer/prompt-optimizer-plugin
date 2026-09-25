# Manual eval — prompt-optimizer

Run each prompt below in a fresh Claude Code session (plugin installed
per Task 6). Record: did `prompt-optimizer` fire (yes/no), which template
was used (3Cs/OPAL/none), did Self-Ask get appended, did the subagent
call any tool (must be "no" — see Global Constraints), and the resulting
optimized prompt text.

| # | Prompt | Expected | Actual |
|---|--------|----------|--------|
| 1 | "schreib was über hunde" | Optimizer fires, 3Cs template, no Self-Ask | |
| 2 | "schreib mir einen Guide zur Pflanzenpflege für Tomaten" | Optimizer fires, OPAL template | |
| 3 | "wie beeinflusst Homeoffice die Teamproduktivität und warum" | Optimizer fires, 3Cs or OPAL + Self-Ask appended | |
| 4 | "You are a marketing expert. Explain in exactly 3 sentences how SEO improves website traffic for a small local bakery." | Optimizer does NOT fire, or fires and returns it near-unchanged | |
| 5 | "ja, mach das so" | Optimizer does NOT fire | |
| 6 | "wie spät ist es in Tokio" | Optimizer does NOT fire | |

## Pass criteria

- Rows 1–3: optimizer fires with the expected template/add-on, and the
  subagent's transcript shows zero tool calls.
- Rows 4–6: optimizer either doesn't fire, or (row 4 only) fires and
  returns the prompt essentially unchanged.
- If any row fails, fix the relevant file (agent description for
  over/under-triggering, SKILL.md decision logic for wrong template
  choice) and rerun that row before considering the eval complete.

## Status

Blocked on a session restart: the plugin was installed via `claude plugin
install` in Task 6, but subagents load at session start, so the
currently-running session cannot dispatch `prompt-optimizer` yet. Run
rows 1–6 in a **new** Claude Code session, fill in the "Actual" column,
then re-run this task's remaining steps (fix any failing row, commit).
