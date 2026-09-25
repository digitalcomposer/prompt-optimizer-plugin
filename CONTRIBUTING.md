# Contributing

Thanks for taking an interest in `prompt-optimizer-plugin`.

## Repo layout

```
.claude-plugin/marketplace.json   marketplace listing (this repo)
plugins/prompt-optimizer/         the plugin itself
  agents/prompt-optimizer.md      the subagent definition (the actual logic)
  README.md                       plugin-level docs & install steps
  EVAL.md                         manual eval log for the subagent
docs/superpowers/                 design spec + implementation plan
```

There is exactly one plugin in this marketplace today. If you're adding a
second plugin, give it its own directory under `plugins/` with its own
`.claude-plugin/plugin.json`, and add an entry to
`.claude-plugin/marketplace.json`.

## Making changes to the agent

The subagent's entire behavior lives in
[`plugins/prompt-optimizer/agents/prompt-optimizer.md`](plugins/prompt-optimizer/agents/prompt-optimizer.md)
— frontmatter (`model`, `tools`, `permissionMode`, `maxTurns`) plus the system
prompt body. There is no separate skill file; the agent is self-contained by
design (a prior version delegated to a skill and occasionally called the
`Skill` tool to reload content already in its own context — see `EVAL.md`).

Before opening a PR that changes the agent's behavior:

1. Bump `version` in both `plugins/prompt-optimizer/.claude-plugin/plugin.json`
   and `.claude-plugin/marketplace.json` (keep them in sync).
2. Add an entry to [`CHANGELOG.md`](CHANGELOG.md).
3. Manually re-run (or extend) the eval table in
   [`plugins/prompt-optimizer/EVAL.md`](plugins/prompt-optimizer/EVAL.md) —
   there is no automated test suite for subagent behavior; a fresh Claude
   Code session with the plugin installed is the test harness.

## Local development

```bash
claude plugin marketplace add /absolute/path/to/prompt-optimizer-plugin
claude plugin install prompt-optimizer@prompt-optimizer-plugin
```

Restart your Claude Code session after any change to the agent file — plugins
load once at session start.

## Pull requests

- Keep PRs focused on one change (one behavior tweak, one doc fix, etc.).
- Describe what you tested (which prompts, what you expected vs. observed).
- CI only validates that the JSON manifests parse — it does not (and cannot)
  test subagent behavior automatically.

## Reporting bugs / suggesting changes

Open a GitHub issue using the provided templates. If it's a security concern,
see [`SECURITY.md`](SECURITY.md) instead of a public issue.
