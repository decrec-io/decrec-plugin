# DecRec

AI explores. You decide. DecRec remembers.

This package is an [Agent Plugin](https://agent-plugins.org): a skill plus the DecRec MCP server. Source: [decrec-io/decrec-plugin](https://github.com/decrec-io/decrec-plugin).

MCP: `https://decrec.io/mcp` (OAuth; no request headers). Plugin installs below connect this except where noted.

## Claude

### Web and Desktop

Same path on both. Skill and MCP come with the plugin.

1. **Customize** → **Plugins** → **Add** → **Add marketplace**
2. URL: `decrec-io/decrec-plugin`
3. **Sync** → select **DecRec** → **Install**
4. Allow when asked.

### Claude Code

```text
/plugin marketplace add decrec-io/decrec-plugin
/plugin install decrec@decrec
```

## ChatGPT

### Desktop (Work)

This path is **Work** only. It does not run in **Chat**.

1. Open **Plugins** → **Add** → **Add a marketplace**.
2. Source: `decrec-io/decrec-plugin`
3. **Add marketplace**, then install **decrec**.
4. Allow when asked. Start a **new Work** conversation.

### Web and Desktop (Chat)

The marketplace plugin does not run here. Add MCP and the skill on [chatgpt.com](https://chatgpt.com). That covers **Web** and Desktop **Chat**.

MCP (tools only — no skill):

1. **Plugins** → **+**
2. Connection: **Server URL**
3. URL: `https://decrec.io/mcp`
4. **Create**. Allow when asked. Start a **new** conversation.

Skill:

1. Download ZIP from the [GitHub repo](https://github.com/decrec-io/decrec-plugin/archive/refs/heads/main.zip)
2. **Plugins** → **Skills** → **+** → upload that zip

### Codex CLI

```text
codex plugin marketplace add decrec-io/decrec-plugin
codex plugin add decrec@decrec
```

## Cursor

Type this in the chat input as a slash command. Do not paste it as a message to the agent.

```text
/add-plugin decrec@https://github.com/decrec-io/decrec-plugin
```

Or **Settings → Customize → Plugins** and install from that GitHub repo.

## Other

Copy [`skills/decrec/*.md`](https://github.com/decrec-io/decrec-plugin/tree/main/skills/decrec) (`SKILL.md` and `explore-prompts.md`) into your client’s skill directory. Add an MCP connector to the MCP URL above.
