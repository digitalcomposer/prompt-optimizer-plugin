## What changed and why

<!-- One or two sentences. Link an issue if there is one. -->

## Testing

<!--
Subagent behavior has no automated test suite (see CONTRIBUTING.md).
List the prompts you tried in a fresh Claude Code session with the plugin
installed, and what you observed vs. expected.
-->

- [ ] Restarted a Claude Code session after installing the changed plugin
- [ ] Tried at least one prompt that should fire (ENHANCE or CLARIFY)
- [ ] Tried at least one prompt that should NOT fire (already clear / trivial)

## Checklist

- [ ] Bumped `version` in `plugins/prompt-optimizer/.claude-plugin/plugin.json`
      **and** `.claude-plugin/marketplace.json` (if behavior changed)
- [ ] Added a `CHANGELOG.md` entry
- [ ] Updated `plugins/prompt-optimizer/EVAL.md` if the eval table is affected
