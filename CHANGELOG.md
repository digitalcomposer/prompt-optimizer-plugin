# Changelog

All notable changes to this project are documented here. Format loosely
follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.2.0] — 2026-09-25

### Changed

- Replaced the agent+skill split (3Cs/OPAL/Self-Ask templates in a separate
  `prompt-optimizing` skill) with a single self-contained agent that
  classifies every request as PASS / ENHANCE / CLARIFY and works uniformly
  across domains, not just content-creation and analytical prompts.
- Rewrote `README.md` (both root and plugin) to describe the new behavior.

### Removed

- `plugins/prompt-optimizer/skills/prompt-optimizing/SKILL.md` — no longer
  used; the agent no longer delegates to a separate skill.

### Fixed

- Removed the underlying cause of an eval-observed bug where the subagent
  would occasionally call the `Skill` tool to reload content already
  present in its own context, violating its "never call a tool" rule (see
  `plugins/prompt-optimizer/EVAL.md`, row 1). There is no separate skill to
  call anymore.

### Repo

- Added `LICENSE` (MIT), `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`,
  `SECURITY.md`, issue/PR templates, and a CI workflow that validates the
  plugin manifests on every push/PR.

## [0.1.0] — 2026-09-25

Initial release: `prompt-optimizer` subagent using 3Cs (default) / OPAL
(structured content-creation) templates, with a conditional Self-Ask
add-on for analytical/multi-factor questions.
