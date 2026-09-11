# rebon-plugins

Plugins for and around [Rebon](https://github.com/Quorafind/Rebon-Releases).

Two different plugin systems live here, and they install in two different
ways. Pick the directory that matches the host you are extending.

| Directory | Host | Installed with |
| --- | --- | --- |
| `claude-code/` | Claude Code | `/plugin marketplace add` |
| `plugins/` | Rebon | `rebon plugin install <path>` |

## Claude Code: `rebon`

Hands a whole turn to Rebon — its own model, its own tools, its own plugins,
its own unattended budget — and reports what it said. Useful when a subtask
stands on its own, should run on a different or cheaper model than the
session you are in, or is long enough that holding it in that context would
crowd it.

```
/plugin marketplace add RebonAI/rebon-plugins
/plugin install rebon@rebon-plugins
```

That gives you two surfaces over the same call:

- `/rebon:delegate <task>` — run it in the current session and read the answer.
- the `delegate` subagent — the same call in the background, visible in the
  agent view.

Both shell out to `rebon exec`, so [Rebon](https://github.com/Quorafind/Rebon-Releases)
must be installed and on `PATH`:

```sh
npm install -g rebon
```

### A subagent per provider

`scripts/sync_claude_plugin_agents.py` in the Rebon repository generates one
subagent per entry in your own `~/.rebon/providers` directory, so a turn can
be handed to any provider you have configured. Those files describe one
machine's provider store rather than the plugin's contents, so they are not
checked in here — run the script against your own checkout if you want them.

## Rebon: `deepseek-responses`

DeepSeek Responses API model provider, running as an `llm/stream` adapter on
Rebon's plugin plane. It owns the whole DeepSeek dialect: request trimming,
`reasoning_text` streaming, custom tool calls, web search events, error and
usage translation.

```sh
git clone https://github.com/RebonAI/rebon-plugins
rebon plugin install ./rebon-plugins/plugins/deepseek-responses
```

`rebon plugin install` takes a local directory path. There is no remote
install path yet, so cloning (or downloading the directory) is the step that
gets the package onto disk.

Then add a provider entry named `deepseek` in `~/.rebon/providers` and select
it. Configuration, models, and the cost notes are in
[`plugins/deepseek-responses/README.md`](plugins/deepseek-responses/README.md).

## Provenance

These directories are copies of `claude-plugin/` and
`plugins/deepseek-responses/` in the Rebon repository, which is where they are
developed. File issues against Rebon.
