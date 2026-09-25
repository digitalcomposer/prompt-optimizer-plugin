# Manual eval — prompt-optimizer

Run each prompt below in a fresh Claude Code session (plugin installed
per Task 6). Record: did `prompt-optimizer` fire (yes/no), which template
was used (3Cs/OPAL/none), did Self-Ask get appended, did the subagent
call any tool (must be "no" — see Global Constraints), and the resulting
optimized prompt text.

| # | Prompt | Expected | Actual |
|---|--------|----------|--------|
| 1 | "schreib was über hunde" | Optimizer fires, 3Cs template, no Self-Ask | Fires. Output: "Schreib einen ansprechenden Text über Hunde in klarer Sprache für ein breites Publikum." Single unlabeled sentence (role+clarity+constraint blended, no explicit background/context clause) — thinner than the shape in SKILL.md §2. No Self-Ask, correct. **Called the `Skill` tool** to load `prompt-optimizing` even though it's declared in the agent's frontmatter and already in context — a real violation of "you never call any tool" (self-confirmed on follow-up: "incorrect... I should have read and applied them directly without calling a tool"). Did not reproduce in rows 2–4. |
| 2 | "schreib mir einen Guide zur Pflanzenpflege für Tomaten" | Optimizer fires, OPAL template | Fires. Clean, correctly labeled Observations/Process/Action/Limitations block. Zero extraneous tool calls (only the mandatory return call). Pass. |
| 3 | "wie beeinflusst Homeoffice die Teamproduktivität und warum" | Optimizer fires, 3Cs or OPAL + Self-Ask appended | Fires. "Kontext: ... / Aufgabe: ..." (2-slot, no separate Constraints line — acceptable, none was inferable) + the exact Self-Ask line appended. Zero extraneous tool calls. Pass. |
| 4 | "You are a marketing expert. Explain in exactly 3 sentences how SEO improves website traffic for a small local bakery." | Optimizer does NOT fire, or fires and returns it near-unchanged | Fires but returns the prompt verbatim, unchanged — exactly the allowed outcome. Zero extraneous tool calls. Pass. |
| 5 | "ja, mach das so" | Optimizer does NOT fire | Not dispatched — matches the agent description's explicit exclusion list. Pass. |
| 6 | "wie spät ist es in Tokio" | Optimizer does NOT fire | Not dispatched — matches the agent description's "simple factual lookup" exclusion. Pass. |

## Pass criteria

- Rows 1–3: optimizer fires with the expected template/add-on, and the
  subagent's transcript shows zero tool calls.
- Rows 4–6: optimizer either doesn't fire, or (row 4 only) fires and
  returns the prompt essentially unchanged.
- If any row fails, fix the relevant file (agent description for
  over/under-triggering, SKILL.md decision logic for wrong template
  choice) and rerun that row before considering the eval complete.

## Status

Run in a fresh session on 2026-09-25 (plugin confirmed enabled via
`claude plugin list`). 5/6 rows pass cleanly. Row 1 has one open issue:
the subagent called the `Skill` tool to load `prompt-optimizing` instead
of relying on the skill content already injected via its frontmatter —
a real "never call a tool" violation that did not reproduce in rows 2–4
(same skill, same declared frontmatter, no Skill call). Likely
probabilistic (haiku sometimes reaches for the tool despite already
having the content) rather than a structural bug in `plugin.json` or the
agent frontmatter.

Suggested fix before considering the eval complete: add an explicit line
to `agents/prompt-optimizer.md`'s "Rules" section — e.g. "The
`prompt-optimizing` skill's content is already included above; never
call the Skill tool to load it" — and rerun row 1 a few times to confirm
the call stops recurring. Not yet applied; awaiting a decision on
whether to tighten the prompt or accept it as within noise.
