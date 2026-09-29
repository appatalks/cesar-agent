# Agent Frontmatter Reference

Last checked against the linked documentation on 2026-09-28. The official references are authoritative if a field or its behavior changes.

These are separate schemas, not one combined format. Copy only the section for the runtime that will load the agent. A field with the same name can have different types or behavior across runtimes.

## Sources

- [GitHub Copilot custom-agent configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [VS Code custom-agent file structure](https://code.visualstudio.com/docs/copilot/customization/custom-agents#_custom-agent-file-structure)
- [Claude Code subagent frontmatter](https://code.claude.com/docs/en/sub-agents#supported-frontmatter-fields)

## GitHub Copilot

These are the fields in GitHub's custom-agent configuration table. `description` is the only required field in that table.

| Field | Type | Required | Behavior and notes |
|---|---|---:|---|
| `description` | string | Yes | Describes the agent's purpose and capabilities. |
| `name` | string | No | Display name. |
| `target` | string | No | `vscode` or `github-copilot`; if omitted, targets both. |
| `tools` | string or list of strings | No | Comma-separated string or YAML list. Omit to enable all available tools; use `[]` to disable all. |
| `model` | string | No | Model to use; if omitted, inherits the default model. |
| `disable-model-invocation` | boolean | No | Defaults to `false`. For Copilot cloud agent, `true` prevents automatic use based on task context. Takes precedence over `infer` when both are set. |
| `user-invocable` | boolean | No | Defaults to `true`. In Copilot cloud agent, `false` prevents manual selection while allowing programmatic access. |
| `infer` | boolean | Retired | Defaults to `true`. Use `disable-model-invocation` and `user-invocable` instead. |
| `mcp-servers` | object | No | Configures additional MCP servers and tools. GitHub's reference says this is not used in VS Code or other IDEs. |
| `metadata` | object of string values | No | Annotation data. GitHub's reference says this is not used in VS Code or other IDEs. |

### VS Code extensions

VS Code documents additional fields for `.agent.md` files. These are not all part of GitHub's supported-field table. In particular, GitHub says `argument-hint` and `handoffs` are ignored by Copilot cloud agent. Do not assume VS Code-only fields work on GitHub.com.

| Field | Type | Behavior and notes |
|---|---|---|
| `description` | string | Brief description shown as placeholder text. The VS Code header itself is optional. |
| `name` | string | Display name; defaults to the file name when omitted. |
| `argument-hint` | string | Hint shown in the chat input. Not supported by Copilot cloud agent. |
| `tools` | list of strings | Built-in tools, tool sets, MCP tools, or extension-contributed tools. `server/*` selects all tools from an MCP server. |
| `agents` | list of strings | Agents allowed as subagents. Use `*` for all or `[]` for none. When specified, include the `agent` tool in `tools`. |
| `model` | string or list of strings | A single model or prioritized model list. If omitted, uses the model selected in the picker. GitHub's table documents a string. |
| `user-invocable` | boolean | Defaults to `true`; controls visibility in the agent dropdown. Subagent invocation is controlled separately. |
| `disable-model-invocation` | boolean | Defaults to `false`; prevents invocation as a subagent when `true`. |
| `infer` | boolean | Deprecated. Use `user-invocable` and `disable-model-invocation` to control the two behaviors independently. |
| `target` | string | `vscode` or `github-copilot`. |
| `mcp-servers` | MCP server configuration | GitHub Copilot target configuration; not used by VS Code's Local harness. The configuration shape is target-specific. |
| `handoffs` | list of objects | Suggested next actions. Each item uses `label` and `agent`; `prompt` is optional. `send` is optional and defaults to `false`; `model` is an optional qualified model name. Not supported by Copilot cloud agent. |
| `hooks` | object | Preview feature for Local-harness agent-scoped hooks. Requires `chat.useHooks` and a trusted workspace. |

The `handoffs` object fields are nested under each list item, not independent top-level frontmatter keys. VS Code documents `label`, `agent`, optional `prompt`, optional `send`, and optional `model`.

### Reasoning effort

The VS Code and GitHub Copilot custom-agent frontmatter references do not define a reasoning-effort field. Requesting high or maximum effort in the Markdown body is a prompt-level instruction, not a platform-enforced setting. Claude Code has a separate `effort` field, but it does not configure these `.github/agents/*.agent.md` profiles.

### GitHub Copilot / VS Code starter

Uncomment and configure only the fields supported by the chosen target. Values below are examples, not defaults.

```yaml
---
description: "Describe the agent's purpose"
# name: my-agent
# target: vscode # or github-copilot; omit to target both
# tools: ["read", "search"]
# model: "model-id"
# disable-model-invocation: false
# user-invocable: true
# infer: true # Retired; prefer the two booleans above
# mcp-servers:
#   my-server:
#     type: local
#     command: "server-command"
# metadata:
#   category: "example"
#
# VS Code-specific fields; check target support before enabling:
# argument-hint: "Describe the task to run"
# agents: ["reviewer"] # or ["*"]; [] disables agent delegation
# handoffs:
#   - label: "Review"
#     agent: "reviewer"
#     prompt: "Review the result"
#     send: false
#     model: "Model Name (vendor)"
# hooks: {}
---

Write the agent instructions here.
```

## Claude Code subagents

Claude Code's file-based subagent format requires `name` and `description`. Field names are case-sensitive; multi-word names use camelCase.

| Field | Type | Required | Behavior and notes |
|---|---|---:|---|
| `name` | string | Yes | Unique identifier. Cannot contain `:` or start with `-`. |
| `description` | string | Yes | Explains when Claude should delegate to this subagent. |
| `tools` | string or list of strings | No | Allowed tools. If omitted, inherits the tools available to subagents. |
| `disallowedTools` | string or list of strings | No | Denylist removed from inherited or specified tools. If both tool fields are set, the denylist is applied first. |
| `model` | string | No | Alias (`sonnet`, `opus`, `haiku`, `fable`), full model ID, or `inherit`. If omitted, Claude Code uses its subagent model order. |
| `permissionMode` | string | No | `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, `plan`, or `manual` (alias for `default`). Ignored for plugin subagents. |
| `maxTurns` | integer | No | Maximum agentic turns before the result is marked partial. |
| `skills` | list of strings | No | Skills to preload into the subagent's context. |
| `mcpServers` | list of server names or inline configurations | No | MCP servers available to the subagent. Ignored for plugin subagents. |
| `hooks` | object | No | Lifecycle hooks scoped to this subagent. Ignored for plugin subagents. |
| `memory` | string | No | Persistent memory scope: `user`, `project`, or `local`. |
| `background` | boolean | No | Set to `true` to keep the subagent in the background when the runtime would otherwise run it in the foreground. |
| `omitClaudeMd` | boolean | No | Set to `true` to omit user, project, and local `CLAUDE.md` files. Managed policy files still load, subject to documented scope exceptions. Requires Claude Code v2.1.271 or later. |
| `effort` | string | No | `low`, `medium`, `high`, `xhigh`, or `max`; availability depends on the model. If omitted, inherits session effort. |
| `isolation` | string | No | Set to `worktree` to run in an isolated temporary Git worktree. |
| `color` | string | No | `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, or `cyan`. |
| `initialPrompt` | string | No | Prepended as the first user turn when this agent runs as the main session agent via `--agent` or the `agent` setting. |
| `experimental` | object | No | File-based only. `cacheTtl` can be `5m` or `1h`; requires Claude Code v2.1.248 or later. Other values are ignored. |

For plugin subagents, `permissionMode`, `mcpServers`, and `hooks` are ignored. Claude Code also ignores unknown frontmatter keys. The `--agents` CLI JSON format is separate from Markdown frontmatter: its `prompt` field supplies the system prompt, and `color` and `experimental` are not accepted there.

### Claude Code starter

Uncomment and configure optional fields as needed. Values below are examples, not defaults.

```yaml
---
name: my-agent
description: "When Claude should delegate to this subagent"
# tools: [Read, Grep, Glob, Bash]
# disallowedTools: [Edit, Write]
# model: inherit
# permissionMode: default
# maxTurns: 20
# skills: [skill-name]
# mcpServers: [server-name]
# hooks: {}
# memory: project
# background: false
# omitClaudeMd: false
# effort: high
# isolation: worktree
# color: cyan
# initialPrompt: "First turn when running as the main session agent"
# experimental:
#   cacheTtl: 5m # or 1h
---

Write the subagent's system prompt here.
```