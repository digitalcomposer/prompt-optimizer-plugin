# Prompt-Optimizer Plugin Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build and locally verify a portable Claude Code plugin (`prompt-optimizer`) whose subagent proactively rewrites vague/unstructured user prompts (3Cs default, OPAL for structured content tasks, conditional Self-Ask) before Claude acts on them, packaged so it installs on any machine via `claude plugin marketplace add` + `claude plugin install`.

**Architecture:** A git repo laid out as a single-plugin marketplace (`.claude-plugin/marketplace.json` at root, `plugins/prompt-optimizer/` containing the plugin). The plugin has one subagent (`agents/prompt-optimizer.md`, model Haiku 4.5, no declared tools) that forwards to one skill (`skills/prompt-optimizing/SKILL.md`) holding the actual rewrite decision logic. No hook, no slash command — the main Claude thread decides proactively, based on the subagent's `description`, when to invoke it.

**Tech Stack:** Claude Code plugin format (JSON manifests + Markdown agent/skill files with YAML frontmatter). No application code, no package manager, no test framework — verification is via `claude plugin validate` (installed CLI, v2.1.282) and a manual eval checklist run live in a Claude Code session.

**Spec:** `docs/superpowers/specs/2026-09-25-prompt-optimizer-plugin-design.md`

## Global Constraints

- Trigger is proactive-subagent only — no UserPromptSubmit hook, no slash command as primary trigger (spec Nicht-Ziele).
- Subagent model: `haiku` (Haiku 4.5) — no `thinking`/`effort` fields in agent frontmatter (not a supported field there; model choice alone drives cost/latency).
- Subagent declares no `tools:` field (spec: must not read files, run commands, or solve the task itself) — enforced primarily through explicit system-prompt instruction, since static validation cannot confirm runtime tool-inheritance behavior (see Task 3).
- Skill must explicitly instruct the rewriter to skip Least-to-Most (no multi-turn escalation available in a single pass) and Temperature (removed/400 on current Claude models — never suggest setting it).
- Subagent's final output is ONLY the rewritten prompt text — no preamble, no meta-commentary, no attempt to solve the underlying task.
- Repo layout is fixed by the spec: `.claude-plugin/marketplace.json` at repo root, plugin under `plugins/prompt-optimizer/`.
- Plugin name: `prompt-optimizer`. Marketplace name: `prompt-optimizer-plugin`. Both version `0.1.0`.
- Fail-open is inherent, not a built component: this is a proactive subagent, not a hard gate in a fixed pipeline. If the `prompt-optimizer` call errors or is never invoked, the main Claude thread simply continues with the original prompt on its own — there is no separate fallback mechanism to build or test for this.

## Review Focus

- **Trivial/conversational messages trigger the subagent anyway** ("ja", "danke", "ok") despite the exclusion list in `description` — a reasonable user expects the optimizer to stay silent on these. Covered by Task 7 eval prompts 5–6.
- **The subagent silently inherits full tool access** despite omitting `tools:` — a reasonable user expects it to never read files or run commands per the spec's scope-guard. Static `claude plugin validate` cannot catch this; Task 3 documents the open question and Task 6/7 add a live behavioral check.
- **`marketplace.json`'s `source` path is wrong or the plugin fails to actually install**, even though the manifest passes schema validation — a reasonable user expects `claude plugin install` to work, not just `claude plugin validate` to pass. Covered by Task 6's live install smoke test.
- **Already-well-structured prompts get needlessly padded** by the rewriter, contradicting SKILL.md §6 — a reasonable user expects a good prompt to come back essentially unchanged. Covered by Task 7 eval prompt 4.
- **The `skills:` reference in the agent frontmatter doesn't match the skill's actual `name:`**, so the subagent never actually loads the rewrite logic and just improvises — a reasonable user expects the documented 3Cs/OPAL/Self-Ask logic to actually run, not generic behavior. Covered by Task 4 (name match check) and Task 7 (behavioral confirmation that OPAL/Self-Ask actually appear when expected).

---

## Task 1: Plugin manifest

**Files:**
- Create: `plugins/prompt-optimizer/.claude-plugin/plugin.json`

**Interfaces:**
- Produces: a valid plugin manifest that Task 2's marketplace entry and Task 6's install both depend on (`name: "prompt-optimizer"`, `version: "0.1.0"`).

- [ ] **Step 1: Confirm the file doesn't exist yet and validation fails**

Run: `claude plugin validate plugins/prompt-optimizer/.claude-plugin/plugin.json`
Expected output (exit code 1):
```
Validating plugin manifest: plugins/prompt-optimizer/.claude-plugin/plugin.json

✘ Found 1 error:

  ❯ file: File not found: plugins/prompt-optimizer/.claude-plugin/plugin.json

✘ Validation failed
```

- [ ] **Step 2: Create the manifest**

Create `plugins/prompt-optimizer/.claude-plugin/plugin.json`:

```json
{
  "name": "prompt-optimizer",
  "version": "0.1.0",
  "description": "Proactively rewrites vague or unstructured prompts using 3Cs/OPAL/Self-Ask techniques before Claude acts on them.",
  "author": {
    "name": "digitalcomposer"
  }
}
```

- [ ] **Step 3: Validate**

Run: `claude plugin validate plugins/prompt-optimizer/.claude-plugin/plugin.json`
Expected output (exit code 0):
```
Validating plugin manifest: plugins/prompt-optimizer/.claude-plugin/plugin.json

✔ Validation passed
```

- [ ] **Step 4: Commit**

```bash
git add plugins/prompt-optimizer/.claude-plugin/plugin.json
git commit -m "Add prompt-optimizer plugin manifest"
```

---

## Task 2: Marketplace manifest

**Files:**
- Create: `.claude-plugin/marketplace.json` (repo root)

**Interfaces:**
- Consumes: `plugins/prompt-optimizer/` (from Task 1) as the `source` path.
- Produces: the marketplace entry Task 6 installs from (`prompt-optimizer@prompt-optimizer-plugin`).

- [ ] **Step 1: Confirm validation fails before the file exists**

Run: `claude plugin validate .claude-plugin/marketplace.json`
Expected output (exit code 1):
```
Validating marketplace manifest: .claude-plugin/marketplace.json

✘ Found 1 error:

  ❯ file: File not found: .claude-plugin/marketplace.json

✘ Validation failed
```

- [ ] **Step 2: Create the marketplace manifest**

Create `.claude-plugin/marketplace.json`:

```json
{
  "name": "prompt-optimizer-plugin",
  "owner": {
    "name": "digitalcomposer"
  },
  "metadata": {
    "description": "Marketplace for the prompt-optimizer Claude Code plugin.",
    "version": "0.1.0"
  },
  "plugins": [
    {
      "name": "prompt-optimizer",
      "description": "Proactively rewrites vague or unstructured prompts using 3Cs/OPAL/Self-Ask techniques before Claude acts on them.",
      "version": "0.1.0",
      "author": {
        "name": "digitalcomposer"
      },
      "source": "./plugins/prompt-optimizer"
    }
  ]
}
```

- [ ] **Step 3: Validate**

Run: `claude plugin validate .claude-plugin/marketplace.json`
Expected output (exit code 0):
```
Validating marketplace manifest: .claude-plugin/marketplace.json

✔ Validation passed
```

- [ ] **Step 4: Commit**

```bash
git add .claude-plugin/marketplace.json
git commit -m "Add prompt-optimizer-plugin marketplace manifest"
```

---

## Task 3: Subagent

**Files:**
- Create: `plugins/prompt-optimizer/agents/prompt-optimizer.md`

**Interfaces:**
- Consumes: the skill name `prompt-optimizing` (must match Task 4's `SKILL.md` frontmatter `name:` exactly — Review Focus item 5).
- Produces: the subagent `prompt-optimizer` that Task 6's install and Task 7's eval invoke.

- [ ] **Step 1: Confirm the agents directory is currently empty of this agent**

Run: `claude plugin validate plugins/prompt-optimizer/agents`
Expected output (exit code 1, directory doesn't exist yet):
```
Validating components in: plugins/prompt-optimizer/agents

✘ Found 1 error:

  ❯ file: File not found: plugins/prompt-optimizer/agents
```

- [ ] **Step 2: Create the subagent**

Create `plugins/prompt-optimizer/agents/prompt-optimizer.md`:

```markdown
---
name: prompt-optimizer
description: Proactively use when the user's raw message is vague, underspecified, lacks explicit format/constraints, or is a substantial task request (content generation, analysis, complex multi-step ask) that would clearly benefit from restructuring before being acted on. Do NOT use for short acknowledgments ("ja", "danke", "mach weiter", "ok"), yes/no replies, simple factual lookups, follow-up questions inside an already-running task, or prompts that already specify role, context, and constraints clearly.
model: haiku
skills:
  - prompt-optimizing
---

You are a thin prompt-rewriting wrapper. Your only job is to rewrite the
user's raw request into a clearer, better-structured prompt and return it —
nothing else.

Rules:
- Use the `prompt-optimizing` skill to decide which template applies and to
  fill it in.
- You never call any tool. You do not read files, run commands, search the
  web, or attempt to answer or solve the user's underlying request. If you
  find yourself reaching for a tool, stop — that is not your job.
- Your final report is ONLY the rewritten prompt text. No preamble such as
  "Here is the optimized prompt:", no explanation of what you changed, no
  markdown code fences around it unless the rewritten prompt itself needs
  them for structure (e.g. a few-shot example block).
- If the input is already clearly structured (role/context and constraints
  already present), return it unchanged or with only a minimal fix — do not
  pad a good prompt to look more "optimized".
```

- [ ] **Step 3: Validate**

Run: `claude plugin validate plugins/prompt-optimizer/agents`
Expected output (exit code 0):
```
Validating components in: plugins/prompt-optimizer/agents

✔ Validation passed
```

- [ ] **Step 4: Note the open verification point**

Static validation confirms the frontmatter schema is well-formed, but it
does **not** confirm whether omitting `tools:` grants zero tools or
inherits the parent's full tool set — that can only be observed at
runtime. Do not treat Step 3's pass as proof of the tools scope-guard.
Task 6 (live install) and Task 7 (eval) must both watch the subagent's
actual transcript for any tool-use block and report back; if it calls a
tool, this task must be reopened to add an explicit tool restriction once
the correct frontmatter field for that is confirmed against the real
schema (check `claude plugin validate --help` output and, if needed,
`claude plugin init test-agent` to inspect a scaffolded template, rather
than guessing a field name).

- [ ] **Step 5: Commit**

```bash
git add plugins/prompt-optimizer/agents/prompt-optimizer.md
git commit -m "Add prompt-optimizer subagent"
```

---

## Task 4: Prompt-optimizing skill

**Files:**
- Create: `plugins/prompt-optimizer/skills/prompt-optimizing/SKILL.md`

**Interfaces:**
- Consumes: nothing from other tasks.
- Produces: the skill `prompt-optimizing` that Task 3's agent references by name — the `name:` field below must read exactly `prompt-optimizing` (Review Focus item 5).

- [ ] **Step 1: Confirm the skills directory doesn't exist yet**

Run: `claude plugin validate plugins/prompt-optimizer/skills`
Expected output (exit code 1):
```
Validating components in: plugins/prompt-optimizer/skills

✘ Found 1 error:

  ❯ file: File not found: plugins/prompt-optimizer/skills
```

- [ ] **Step 2: Create the skill**

Create `plugins/prompt-optimizer/skills/prompt-optimizing/SKILL.md`:

```markdown
---
name: prompt-optimizing
description: Rewrite a raw, unstructured prompt into a clearer, better-structured one before it is acted on. Applies the 3Cs framework by default, OPAL for structured content-creation tasks, and a conditional Self-Ask instruction for analytical/multi-factor questions. Use when asked to optimize, rewrite, tighten, or restructure a prompt.
---

# Prompt Optimizing

Decision procedure for rewriting a raw prompt. Follow these steps in order.

## 1. Classify the task type

- **Structured content-creation task** — the request asks to produce a
  document/guide/report/article with recognizable phases (introduction,
  steps, conclusion; e.g. "write a guide to X", "create a report on Y") →
  use the **OPAL template** (§3).
- **Everything else** (default) → use the **3Cs template** (§2).

## 2. 3Cs template (default)

Fill these three slots from what the raw prompt implies — never invent
facts or specifics the user didn't state or clearly imply:

- **Context** — what background/complexity level should the model assume?
  Keep it concise (1-2 sentences).
- **Clarity** — state the concrete intent in one unambiguous sentence.
  Resolve vague verbs ("look at", "handle") into a specific action.
- **Constraints** — output format, length, style, or role, if inferable or
  if a sane default improves the answer (e.g. a "write X" request with no
  stated length benefits from a rough word/length constraint).

Assemble into a single prompt, in this shape:

```
[Role, if one is implied or clearly useful]
[Context: 1-2 sentences]
[Clarity: the concrete task, one sentence]
[Constraints: format/length/style, if any]
```

## 3. OPAL template (structured content-creation tasks)

Fill the four slots, again only from what's inferable — do not invent
domain facts:

- **Observations** — background/context the model should know before
  starting.
- **Process** — the phases/structure the output should follow.
- **Action** — the concrete output type and style/tone.
- **Limitations** — what to avoid or exclude.

Assemble as:

```
Observations: "..."
Process: "..."
Action: "..."
Limitations: "..."
```

## 4. Self-Ask add-on (conditional)

If the request is analytical or multi-factor — it asks "why", "how does X
affect Y", asks for a comparison, or asks to explain a causal chain or a
question that plausibly has several contributing factors — append this
line to the assembled prompt (after the 3Cs or OPAL block):

```
Zerlege die Frage zuerst in Teilfragen, beantworte jede einzeln,
synthetisiere dann die Endantwort.
```

Do not add this for simple, single-fact, or single-step requests.

## 5. Explicitly excluded techniques

- **Least-to-Most prompting** — do not apply. It requires seeing a
  response and escalating across multiple turns; a single-pass rewrite
  has no result to react to.
- **Temperature** — do not mention or suggest setting it. On current
  Claude models (Sonnet 5, Opus 5/5.5, Fable 5/5.1, Opus 4.7/4.8) the
  `temperature` API parameter is removed and returns HTTP 400; it is not
  a lever available to the rewriter or the model being asked.

## 6. Already well-structured prompts

If the raw prompt already states role/context and constraints clearly,
return it unchanged, or with only a minimal, targeted fix (e.g. resolving
one ambiguous word). Do not restructure or lengthen a prompt that doesn't
need it.

## 7. Output contract

Return ONLY the rewritten prompt text. No preamble, no explanation of what
changed, no meta-commentary, and do not attempt to answer or solve the
task the prompt describes.
```

- [ ] **Step 3: Validate**

Run: `claude plugin validate plugins/prompt-optimizer/skills`
Expected output (exit code 0):
```
Validating components in: plugins/prompt-optimizer/skills

✔ Validation passed
```

- [ ] **Step 4: Confirm the name match**

Run: `grep "^name:" plugins/prompt-optimizer/agents/prompt-optimizer.md plugins/prompt-optimizer/skills/prompt-optimizing/SKILL.md`
Expected: the agent's `skills:` entry (`prompt-optimizing`) matches this
skill's `name:` field exactly. If the strings differ, fix whichever file
is wrong before continuing — this is Review Focus item 5.

- [ ] **Step 5: Commit**

```bash
git add plugins/prompt-optimizer/skills/prompt-optimizing/SKILL.md
git commit -m "Add prompt-optimizing skill"
```

---

## Task 5: READMEs

**Files:**
- Create: `plugins/prompt-optimizer/README.md`
- Create: `README.md` (repo root)

**Interfaces:**
- Consumes: the marketplace name (`prompt-optimizer-plugin`) and plugin name (`prompt-optimizer`) fixed in Tasks 1–2.
- Produces: install instructions Task 6 will execute verbatim to confirm they're correct.

- [ ] **Step 1: Write the plugin README**

Create `plugins/prompt-optimizer/README.md`:

```markdown
# prompt-optimizer

Proactively rewrites vague or unstructured prompts before Claude acts on
them, using three prompting techniques:

- **3Cs** (Context/Clarity/Constraints) — default template.
- **OPAL** (Observations/Process/Action/Limitations) — for structured
  content-creation tasks ("write a guide to X").
- **Self-Ask** — appended conditionally for analytical/multi-factor
  questions.

It deliberately does not use Least-to-Most prompting (needs a multi-turn
feedback loop) or Temperature (not a usable API parameter on current
Claude models).

## How it triggers

There is no hook and no slash command. The main Claude thread proactively
decides — based on the `prompt-optimizer` subagent's description — whether
your message would benefit from restructuring. Short acknowledgments,
follow-up questions, and already well-structured prompts are excluded by
design.

## What you'll see

When it fires, Claude shows a short line ("Prompt optimiert zu: ...")
before continuing — it never silently swaps your prompt without telling
you.

## Install

Local development / same machine:

```bash
claude plugin marketplace add /path/to/prompt-optimizer-plugin
claude plugin install prompt-optimizer@prompt-optimizer-plugin
```

From GitHub, on any machine with Claude Code installed:

```bash
claude plugin marketplace add <github-owner>/<repo>
claude plugin install prompt-optimizer@prompt-optimizer-plugin
```

Verify it's active:

```bash
claude plugin list
```
```

- [ ] **Step 2: Write the repo root README**

Create `README.md`:

```markdown
# prompt-optimizer-plugin

Single-plugin Claude Code marketplace. See
[`plugins/prompt-optimizer/README.md`](plugins/prompt-optimizer/README.md)
for what the plugin does and how to install it.

## Repo layout

```
.claude-plugin/marketplace.json   marketplace listing (this repo)
plugins/prompt-optimizer/         the plugin itself
docs/superpowers/specs/           design spec
docs/superpowers/plans/           implementation plan
```
```

- [ ] **Step 3: Confirm the install commands are copy-paste accurate**

Run: `grep -A2 "claude plugin marketplace add /path/to" plugins/prompt-optimizer/README.md`
Expected: the marketplace/plugin names in the shown commands
(`prompt-optimizer-plugin`, `prompt-optimizer`) match `name:` in
`.claude-plugin/marketplace.json` and `plugins/prompt-optimizer/.claude-plugin/plugin.json` exactly (Task 6 will run these commands verbatim — a mismatch here would surface there, but catching it now saves a cycle).

- [ ] **Step 4: Commit**

```bash
git add README.md plugins/prompt-optimizer/README.md
git commit -m "Add plugin and repo READMEs"
```

---

## Task 6: Local install smoke test

**Files:** none created — this task exercises Tasks 1–5's output through the real `claude plugin` CLI.

**Interfaces:**
- Consumes: the full plugin directory from Tasks 1–4 and the exact commands documented in Task 5's `plugins/prompt-optimizer/README.md`.

⚠️ This task registers a local marketplace and installs a plugin under
your **user** scope in this machine's real Claude Code configuration
(`~/.claude/plugins/known_marketplaces.json` and the installed-plugins
list). It is reversible (`claude plugin uninstall` / `claude plugin
marketplace remove`), but it does change shared, persistent state outside
this repo — tell the user what you're about to run before running it if
you're a subagent executing this task unattended.

- [ ] **Step 1: Register the local marketplace**

Run (from the repo root):
```bash
claude plugin marketplace add .
```
Expected: confirmation that a marketplace named `prompt-optimizer-plugin`
was added, pointing at this repo's path.

- [ ] **Step 2: Install the plugin**

Run:
```bash
claude plugin install prompt-optimizer@prompt-optimizer-plugin
```
Expected: install succeeds, names version `0.1.0`.

- [ ] **Step 3: Confirm it's listed**

Run:
```bash
claude plugin list
```
Expected: `prompt-optimizer` appears in the output as installed and
enabled.

- [ ] **Step 4: Restart the Claude Code session**

Plugins load at session start. Note in your final report to the user
that a new session (or `/restart` if the harness supports it) is required
before the `prompt-optimizer` subagent becomes available for Task 7.

- [ ] **Step 5: Record the result**

No commit needed (no files changed) — report the outcome of Steps 1–4
(pass/fail, exact error text if any) in the task's completion notes for
the next task to pick up.

---

## Task 7: Manual eval

**Files:**
- Create: `plugins/prompt-optimizer/EVAL.md`

**Interfaces:**
- Consumes: the live installed plugin from Task 6 (must be run in a fresh session where the plugin is active).

- [ ] **Step 1: Write the eval checklist**

Create `plugins/prompt-optimizer/EVAL.md`:

```markdown
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
```

- [ ] **Step 2: Run all 6 prompts in a fresh session and fill in the "Actual" column**

Use the freshly restarted session from Task 6. For each prompt, note
whether the subagent fired, which template it used, and whether it called
any tool — this is the live confirmation for Review Focus items 1, 2, and
5 that static validation couldn't provide.

- [ ] **Step 3: Fix any failing rows**

If a row's "Actual" doesn't match "Expected", edit the failing file
(most likely `agents/prompt-optimizer.md`'s `description` for
trigger-boundary misses, or `skills/prompt-optimizing/SKILL.md` for
wrong-template misses) and rerun only that row.

- [ ] **Step 4: Commit**

```bash
git add plugins/prompt-optimizer/EVAL.md
git commit -m "Add manual eval results for prompt-optimizer"
```
