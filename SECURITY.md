# Security Policy

This is a Claude Code plugin: a subagent definition (a markdown system
prompt) plus a JSON manifest. It contains no server, no network calls of its
own, and no dependencies — but it does run with whatever tool permissions
you grant the subagent (currently: `Read` only, see
[`plugins/prompt-optimizer/agents/prompt-optimizer.md`](plugins/prompt-optimizer/agents/prompt-optimizer.md)).

## Reporting a vulnerability

Please report security concerns (e.g. a prompt-injection path that gets the
subagent to exceed its declared tool permissions, or a manifest that could be
abused during plugin install) using GitHub's private reporting flow rather
than a public issue:

**[Report a vulnerability](https://github.com/digitalcomposer/prompt-optimizer-plugin/security/advisories/new)**
(repo → Security tab → "Report a vulnerability").

You should get an initial response within a few days. If the report is
confirmed, a fix will be released and credited in the advisory unless you
ask to stay anonymous.

## Supported versions

Only the latest published version of the `prompt-optimizer` plugin is
supported. There is no long-term-support branch.
