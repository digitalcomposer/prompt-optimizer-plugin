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
claude plugin marketplace add digitalcomposer/prompt-optimizer-plugin
claude plugin install prompt-optimizer@prompt-optimizer-plugin
```

Verify it's active:

```bash
claude plugin list
```
