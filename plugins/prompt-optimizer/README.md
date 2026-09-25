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

## Installation (step by step, no prior plugin experience needed)

### Prerequisite: Claude Code itself

You need the Claude Code CLI installed first. Check whether you already
have it:

```bash
claude --version
```

If that prints a version number (e.g. `2.1.282 (Claude Code)`), skip to
Step 1. If you get "command not found", install Claude Code first — see
the official docs at [docs.claude.com](https://docs.claude.com) (native
installer) or, if you have Node.js, `npm install -g @anthropic-ai/claude-code`.
Come back here once `claude --version` works.

### Step 1 — Add this plugin's marketplace

A "marketplace" is just a source Claude Code can install plugins from —
in this case, this GitHub repo. Run:

```bash
claude plugin marketplace add digitalcomposer/prompt-optimizer-plugin
```

You should see something like:

```
Adding marketplace…SSH not configured, cloning via HTTPS: https://github.com/digitalcomposer/prompt-optimizer-plugin.git
Refreshing marketplace cache (timeout: 120s)…
Cloning repository (timeout: 120s): https://github.com/digitalcomposer/prompt-optimizer-plugin.git
Clone complete, validating marketplace…
✔ Successfully added marketplace: prompt-optimizer-plugin (declared in user settings)
```

(This step only registers the source — it does not install anything yet.)

### Step 2 — Install the plugin

```bash
claude plugin install prompt-optimizer@prompt-optimizer-plugin
```

Expected output:

```
Installing plugin "prompt-optimizer@prompt-optimizer-plugin"...✔ Successfully installed plugin: prompt-optimizer@prompt-optimizer-plugin (scope: user)
```

### Step 3 — Confirm it installed

```bash
claude plugin list
```

Look for an entry like this in the output:

```
❯ prompt-optimizer@prompt-optimizer-plugin
  Version: 0.1.0
  Scope: user
  Status: ✔ enabled
```

### Step 4 — Restart Claude Code

Plugins (and the subagent inside this one) only load when a Claude Code
session starts. If you installed it while a session was already running,
close that session and open a new one (or use your harness's restart
command, e.g. `/restart` if it has one) before continuing.

### Step 5 — Try it

In a fresh session, type a deliberately vague prompt, for example:

```
schreib was über hunde
```

If it's working, Claude will briefly show a line like "Prompt optimiert
zu: ..." before continuing — that's the `prompt-optimizer` subagent
having rewritten your request. It won't fire on short replies like "ja"
or "danke", or on prompts that are already clear and well-structured —
that's by design, see "How it triggers" above.

### Troubleshooting

- **`claude: command not found`** — Claude Code itself isn't installed
  yet; see Prerequisite above.
- **`claude plugin marketplace add .` fails** with "Invalid marketplace
  source format" — a bare `.` isn't accepted. Use the GitHub form above,
  or an absolute local path (`claude plugin marketplace add /full/path/to/prompt-optimizer-plugin`)
  if you're developing locally instead of installing from GitHub.
- **The plugin doesn't appear to do anything** — make sure you restarted
  your session after installing (Step 4). Also remember it's proactive,
  not automatic-on-every-message: trivial or already well-structured
  prompts won't trigger it.
- **Uninstalling:**
  ```bash
  claude plugin uninstall prompt-optimizer
  claude plugin marketplace remove prompt-optimizer-plugin
  ```
