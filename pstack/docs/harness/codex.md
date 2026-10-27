# pstack in Codex

pstack skills use Cursor names. In Codex, read this file once per session, then map each Cursor name in a skill with the tables below. `<pstack>` means the folder that holds `skills/` and `docs/`. If a Cursor field has no row, drop it and say so. Never pass a field or value the tool schema rejects.

## Tools

| Cursor | Codex |
| --- | --- |
| `Task` | `spawn_agent` |
| `AskQuestion` | `request_user_input` when the session offers it. Otherwise ask in plain text. |
| Open a todolist | Use the session's plan tool when available. Otherwise keep the list in your reply and update it at each phase. |
| Run another pstack skill (`/name` or "the **name** skill") | Read `<pstack>/skills/<name>/SKILL.md` and follow it. When you tell the user to type it, write `$pstack:<name>`. |
| `Grep`, `Glob` | Use the session's shell tool with `rg`. |

## Subagent fields

| Cursor | Codex |
| --- | --- |
| `subagent_type: generalPurpose` | `agent_type: "default"` |
| `poteto-agent`, `comment-sicko` | Spawn a default agent and tell it to read `<pstack>/agents/<name>.md` and this adapter. Give poteto-agent the absolute path to `<pstack>/skills/poteto-mode/SKILL.md`. |
| `readonly: true` | `agent_type: "explorer"`. Its sandbox is inherited, so the brief also says "do not edit files", and you check `git status` afterward. |
| `readonly: false` | `default`. It keeps the session's MCP servers. |
| `environment: "cloud"` | Prepare a separate git worktree and tell the worker to work at its absolute path. Name the branch or commit the worker checks out. |
| `environment: "local"` | Omit. |
| `cloud_base_branch: <b>` | Use `<b>` as the worktree's starting revision. |
| `run_in_background: true` | Omit. Subagents run in the background by default. Collect results with the agent wait tool. |
| `resume` | Use the session's subagent follow-up tool with the existing agent ID. Reuse only under poteto-mode's Subagents rules; fresh subagents remain the default. |
| `file_attachments` | List the paths in the brief. |

## Models

| Cursor | Codex |
| --- | --- |
| `claude-<family>-<version>-<effort>[-fast]`, such as `claude-opus-5-5-xhigh` | Omit `model` so the agent runs on the session model, and say so. |
| A `grok-*`, `gpt-*`, or other non-Claude slug | For a detected Codex model, pass `model` and `reasoning_effort` separately. Otherwise omit `model` so the agent runs on the session model, and say so. |
| `auto` or `inherit-parent` | Omit model and reasoning overrides. |
| The `pstack-models.mdc` rule, `~/.cursor/rules/pstack-models.mdc` | `${CODEX_HOME:-$HOME/.codex}/pstack-models.md`. Read it when loading this adapter; Codex does not load it as a rule. If the file or the role's line is missing, use the skill's default slug. |
| A role value `<model> <effort>`, such as `opus high` | Use detected Codex values as `model: "<model>"` and `reasoning_effort: "<effort>"`. |

## setup-pstack

Codex does not load the model preference file as a rule, so this adapter reads it. Follow `setup-pstack` with these changes:

- Detect. The detected models are the `model` values that the `spawn_agent` tool schema lists, each with a supported `reasoning_effort`. Write a real model as `<model> <effort>`. The effort is the last token, so the budget step works as written.
- Defaults. Map each default slug with the rows above: both `claude-opus-5-5-xhigh` and `grok-4.7-xhigh-fast` are `inherit-parent`.
- Write. Write `${CODEX_HOME:-$HOME/.codex}/pstack-models.md` with no frontmatter, then the comment lines, the `# budget` line, and one line per role from step 5. Overwrite the whole file. Never put the role lines in `AGENTS.md`.

## Paths

| Cursor | Codex |
| --- | --- |
| `.cursor/skills/`, `~/.cursor/skills/` | `.agents/skills/`, `~/.agents/skills/` |
| `~/.cursor/plugins/`, `.cursor-plugin/plugin.json` | `${CODEX_HOME:-$HOME/.codex}/plugins/cache/`, `plugin.json` |
| `.cursor/rules/*.mdc`, always-applied rules | `AGENTS.md`. Codex never reads `.mdc` files. The model rule is the exception. See Models. |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | `<pstack>/<path>` when that file exists. The plugin ships the skills, `docs/guide/`, and `README.md`. Read the local copy and never fetch it. Give the user the URL when they need a link. |
| Transcripts in `~/.cursor/projects/<slug>/agent-transcripts/` | Use the current thread ID with the session's thread tools. For local JSONL in `${CODEX_HOME:-$HOME/.codex}/sessions/`, filter `session_meta.payload.cwd` to this workspace first. Read only this workspace's transcripts. |

## Cursor-only features

Codex has no Cursor Automations, Bugbot, Cursor cloud agents, or built-in PR tool. When a step needs one, put one line in your reply, `Skipped in Codex: <step>. <reason>.`, then go on to the next step. Do not stop, ask, or retry. When a later step needs what the skipped step made, such as a URL or a key, leave a placeholder and name it in the line.

| Cursor | Codex |
| --- | --- |
| Cursor Automations: routines, `update_state` routine calls, `api2.cursor.sh/automations` webhooks, routine wakes, and `SendToUser` secret requests | Skip with the line. |
| Bugbot | Bugbot comments on a GitHub PR are ordinary review threads, so triage them with the forge command that the step gives. Skip a step that starts, waits for, or configures Bugbot when the PR has no Bugbot check or comment. |
| Cursor cloud agents | `environment: "cloud"` maps to a worktree, as in Subagent fields. Skip a step that needs the cloud agent itself, such as its dashboard status or its own machine. |
| A built-in PR tool | Use the forge command that the step gives, such as `gh pr create`. That is not a skip, so write no line. |
