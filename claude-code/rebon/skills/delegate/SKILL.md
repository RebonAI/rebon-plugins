---
name: delegate
description: Hand a self-contained task to Rebon, a separate agent harness with its own models, plugins and unattended budget. Use when a subtask stands on its own, when it should run on a different or cheaper model than this session, or when it is long enough that holding it here would crowd the context.
---

# Delegating a turn to Rebon

Rebon runs a full agent loop of its own: its own model, its own tools, its own
plugins, its own transcript. It is spawned as a child process, so **it cannot
see this conversation**. Everything it needs must be written into the prompt.

## The call

```bash
log=$(mktemp); rebon exec "$TASK" 2>"$log"; grep -m1 '^session ' "$log"
```

- **stdout is Rebon's answer and nothing else.** Report it as the result.
- Everything else — traces, the session id — goes to stderr, which is why the
  recipe sends it to a file and pulls the session id back out afterwards. The
  session id is what makes a follow-up possible, so keep it and show it.
- Rebon runs in the current directory. Pass `--cwd <dir>` if that is not the
  project root.

## Writing the prompt

A prompt that says "fix the bug we discussed" gives Rebon nothing to work with.
Write the goal, the files, the constraints, and what "done" looks like. Assume
the reader has never seen this conversation — because it has not.

## Choosing the model

```bash
rebon --provider <name> --model <id> exec "$TASK"
```

Unset, Rebon uses the model from its own config. Reach for these when running
on a different model is the *reason* to delegate.

## Bounding the run

| flag | effect |
|---|---|
| `--max-duration <seconds>` | hard ceiling; the run stops and reports `stopReason=max_duration` |
| `--max-iterations <n>` | stop after n model iterations |
| `--verify-rounds <n>` | make it re-check its own work n times before accepting "done" |
| `--effort <level>` | thinking level for this run |

Pass `--max-duration 300` unless the task is clearly longer than that, and keep
it under your Bash tool timeout. If the call is killed from this side, Rebon's
session is left mid-turn. For anything longer, run it in the background and
read the session back afterwards.

## Continuing the same session

```bash
log=$(mktemp); rebon exec --resume <session-id> "$FOLLOW_UP" 2>"$log"; grep -m1 '^session ' "$log"
```

The resumed session keeps its history, so a follow-up does not have to restate
the original task.

## The other form: the `rebon:delegate` subagent

The plugin also registers a subagent under this name. Same driver, but it runs in
its own context, so Rebon's output never lands in this conversation — at the cost
of one extra model round trip for the subagent itself. Reach for the subagent
when the answer would be long, and run the command here when it would not.

## Before you delegate

- **Rebon does not ask permission.** `rebon exec` is the unattended surface: it
  approves its own tool calls. Delegating authorises those edits and commands to
  run without a prompt, so do not hand it something you would have stopped.
- **Every call re-sends Rebon's whole system prompt** — roughly 20k input tokens
  before any work happens. A task you could finish here in two tool calls is
  cheaper done here.
