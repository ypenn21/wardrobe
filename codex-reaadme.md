# Codex Configuration Guide

Codex configuration is layered. On this machine, the main Codex home is:

```text
/Users/yannipeng/.codex
```

## Configuration map

| What | User-wide location | Repository location |
| --- | --- | --- |
| Main configuration | `~/.codex/config.toml` | `<repo>/.codex/config.toml` |
| Persistent instructions | `~/.codex/AGENTS.md` | `<repo>/AGENTS.md` |
| Custom subagent definitions | `~/.codex/agents/*.toml` | `<repo>/.codex/agents/*.toml` |
| Skills | `~/.agents/skills/<skill>/SKILL.md` | `<repo>/.agents/skills/<skill>/SKILL.md` |
| MCP configuration | `[mcp_servers.*]` in `config.toml` | Same, in trusted project config |
| Profiles | `~/.codex/<profile>.config.toml` | User-level only |

## 1. Main Codex configuration

The current user configuration is:

```text
~/.codex/config.toml
```

Use it for settings that should apply everywhere:

```toml
model = "gpt-5.6-sol"
model_reasoning_effort = "high"
approval_policy = "on-request"
sandbox_mode = "workspace-write"

[agents]
max_threads = 6
max_depth = 1
```

For settings that belong only to Wardrobe, create:

```text
/Users/yannipeng/git-projects/wardrobe/.codex/config.toml
```

Project configuration is loaded only when the repository is trusted. Nested `.codex/config.toml` files can override settings for subdirectories.

One-off overrides can be passed through the CLI:

```sh
codex --model gpt-5.4
codex --config model_reasoning_effort='"high"'
```

## 2. Where subagents live

Subagent execution threads are runtime objects; they are not folders containing checked-out copies of an agent.

Their reusable definitions live here:

```text
~/.codex/agents/*.toml
<repo>/.codex/agents/*.toml
```

This repository already has a custom agent:

```text
.codex/agents/gemini-engineer.toml
```

A basic custom agent looks like:

```toml
name = "reviewer"
description = "Review code for correctness, security, and missing tests."
developer_instructions = """
Inspect changes without editing files.
Return findings with file and line references.
"""
model = "gpt-5.6-sol"
model_reasoning_effort = "high"
sandbox_mode = "read-only"
```

Built-in roles include `default`, `worker`, and `explorer`.

Global limits go in `config.toml`:

```toml
[agents]
max_threads = 6
max_depth = 1
```

You can explicitly request subagents with prompts such as:

```text
Use three subagents: one for security, one for test gaps, and one for maintainability.
```

In the CLI, `/agent` or `/subagents` lets you inspect active threads. See the official [Subagents documentation](https://learn.chatgpt.com/docs/agent-configuration/subagents).

## 3. Where MCP servers live

There are two separate things:

1. The MCP server definition lives in `config.toml`.
2. The actual server is either a local executable or a remote HTTP service.

Local server example:

```toml
[mcp_servers.my_server]
command = "node"
args = ["/absolute/path/to/server.mjs"]
startup_timeout_sec = 30

[mcp_servers.my_server.env]
MY_SETTING = "value"
```

Remote server example:

```toml
[mcp_servers.docs]
url = "https://example.com/mcp"
```

The current user configuration defines MCP servers under:

```toml
[mcp_servers.node_repl]
[mcp_servers.computer-use]
```

The `node_repl` executable currently comes from the ChatGPT application bundle:

```text
/Applications/ChatGPT.app/Contents/Resources/cua_node/bin/node_repl
```

MCP servers do not all live in one special MCP folder. The configuration says how Codex launches or contacts each server. See the official [MCP documentation](https://learn.chatgpt.com/docs/extend/mcp).

## 4. Where instructions live

There are several instruction layers with different ownership.

### OpenAI system instructions

The base system instructions are supplied by the Codex product/runtime. They are not ordinarily stored in the repository or exposed as an editable local prompt file.

### Durable personal instructions

Put these in:

```text
~/.codex/AGENTS.md
```

These apply across repositories.

### Repository instructions

Put shared project rules in:

```text
<repo>/AGENTS.md
```

Codex walks from the repository root toward the current working directory. A closer nested `AGENTS.md` takes precedence for files in that subtree.

You can also use `AGENTS.override.md` when you deliberately need to replace the normal instructions at a particular level.

### Subagent instructions

Put role-specific instructions in the agent's TOML file:

```toml
developer_instructions = """
Your instructions here.
"""
```

### Custom model instruction file

For a dedicated instruction document, `config.toml` also supports a `model_instructions_file` path. In project configuration, relative paths are resolved relative to the containing `.codex` directory.

For most repository guidance, prefer `AGENTS.md`. See [Custom instructions with AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md).

## 5. Where skills live

The recommended locations are:

```text
# Repository skills
<repo>/.agents/skills/<skill-name>/SKILL.md

# Personal skills
~/.agents/skills/<skill-name>/SKILL.md

# Administrator-installed skills
/etc/codex/skills/<skill-name>/SKILL.md

# Built-ins
Bundled with Codex
```

This repository already has:

```text
.agents/skills/generate-outfits/
.agents/skills/import-clothes/
```

Each skill must contain at least:

```text
my-skill/
└── SKILL.md
```

Example:

```md
---
name: my-skill
description: Use when the user asks Codex to perform a particular workflow.
---

# Instructions

Follow these steps...
```

A skill can also contain:

```text
my-skill/
├── SKILL.md
├── agents/openai.yaml
├── scripts/
├── references/
└── assets/
```

## Choosing the right extension point

- `AGENTS.md`: How Codex should behave in a repository.
- Custom agent: A specialized worker persona and configuration.
- Skill: A reusable workflow Codex loads when relevant.
- MCP: Live tools or external data and actions.
- Plugin: A distributable bundle that can contain skills, MCP configuration, apps, hooks, and assets.
