# prompt-optimizer-plugin

[![Validate](https://github.com/digitalcomposer/prompt-optimizer-plugin/actions/workflows/validate.yml/badge.svg)](https://github.com/digitalcomposer/prompt-optimizer-plugin/actions/workflows/validate.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-plugin-cc785c)](https://docs.claude.com/en/docs/claude-code)

A [Claude Code](https://docs.claude.com/en/docs/claude-code) plugin
marketplace containing **`prompt-optimizer`**: a subagent that proactively
rewrites vague or unstructured prompts into minimal, precise task
instructions — before Claude acts on them — without changing your intent.

See [`plugins/prompt-optimizer/README.md`](plugins/prompt-optimizer/README.md)
for the full behavior spec and step-by-step install instructions.

## Quickstart

```bash
claude plugin marketplace add digitalcomposer/prompt-optimizer-plugin
claude plugin install prompt-optimizer@prompt-optimizer-plugin
```

Restart your Claude Code session, then try a deliberately vague prompt (e.g.
`schreib was über hunde`) — see the plugin README for what to expect.

## Repo layout

```
.claude-plugin/marketplace.json   marketplace listing (this repo)
plugins/prompt-optimizer/         the plugin itself
  agents/prompt-optimizer.md      the subagent (its whole behavior lives here)
  README.md                       plugin docs & install steps
  EVAL.md                         manual eval log
docs/superpowers/specs/           design spec
docs/superpowers/plans/           implementation plan
```

## Contributing

Bug reports and feature requests: use the issue templates. Security
concerns: see [`SECURITY.md`](SECURITY.md), not a public issue. Development
workflow and PR checklist: see [`CONTRIBUTING.md`](CONTRIBUTING.md). This
project follows the [Contributor Covenant](CODE_OF_CONDUCT.md).

## License

[MIT](LICENSE) — see [`CHANGELOG.md`](CHANGELOG.md) for release history.

---

*Keywords: Claude Code plugin, Claude Code subagent, prompt engineering,
prompt optimizer, prompt rewriting, LLM agent, Anthropic Claude.*
