# Rebon Plugins

The publishing repository for plugins built by the Rebon team.

Rebon is an agent harness — [reboncode.ai](https://reboncode.ai).

This repository is a distribution channel, not a workspace. The plugins here
are developed in Rebon's own repository and copied out on release, so issues
and pull requests are closed and only first-party plugins are published here.
Report anything you find through [reboncode.ai](https://reboncode.ai).

Two different plugin systems live here, and they install in two different
ways. Pick the directory that matches the host you are extending.

| Directory | Host | Installed with |
| --- | --- | --- |
| `claude-code/` | Claude Code | `/plugin marketplace add` |
| `plugins/` | Rebon | `rebon plugin install <path>` |

## Claude Code: `rebon`

Hands a task to Rebon as a background job — its own model, its own tools,
its own plugins, its own unattended budget — and brings the outcome back to
the session. Useful when a subtask stands on its own, should run on a
different or cheaper model than the session you are in, or is long enough
that holding it in that context would crowd it.

```
/plugin marketplace add RebonAI/rebon-plugins
/plugin install rebon@rebon-plugins
```

The plugin declares an MCP server, `rebon mcp serve`, so Rebon 1.2.0 or later
must be installed and on `PATH`:

```sh
npm install -g rebon
```

That gives you two surfaces over the same server:

- `/rebon:delegate <task>` — start the job from the current session.
- the `delegate` subagent — the same start in the background, visible in the
  agent view.

`exec_start` returns a job id at once. The job belongs to Rebon's background
supervisor rather than to Claude Code, so no Bash timeout cuts it off and
closing the session leaves it running; `rebon agents` and `rebon attach` show
it like any other job.

When the job finishes, or stops to ask a question or for permission, the
server pushes a message back into the session. Claude Code only takes those
pushes from a plugin named at start-up:

```sh
claude --dangerously-load-development-channels plugin:rebon@rebon-plugins
```

Without the flag every tool still works; ask Claude to check the job, and
`job_status` answers what the push would have said.

### A subagent per provider

Rebon ships a script that generates one subagent per entry in your own
`~/.rebon/providers` directory, so a turn can be handed to any provider you
have configured. Those files describe one machine's provider store rather
than the plugin's contents, so they are not published here.

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
install path yet, so cloning — or downloading the directory — is the step
that gets the package onto disk.

Then add a provider entry named `deepseek` in `~/.rebon/providers` and select
it. Configuration, models, and the cost notes are in
[`plugins/deepseek-responses/README.md`](plugins/deepseek-responses/README.md).

## License

Licensed under either of [Apache License, Version 2.0](LICENSE-APACHE) or
[MIT license](LICENSE-MIT) at your option. Rebon itself is not open source;
these plugins are.

`plugins/deepseek-responses` reproduces material from DeepSeek Harness under
its own MIT license — see the `NOTICE` and `LICENSE-DEEPSEEK-HARNESS` beside
it.
