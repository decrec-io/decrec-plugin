# DecRec

Decision memory for people who move fast.

This package is an [Agent Plugin](https://agent-plugins.org): a skill plus the DecRec MCP server. Source: [decrec-io/decrec-skill](https://github.com/decrec-io/decrec-skill).

**MCP** (OAuth; no request headers):

- URL: `https://decrec.io/mcp`

Add the URL as a remote MCP server / custom connector. The client runs OAuth; you sign in on DecRec and Allow.

## Claude

Install the plugin from [decrec-io/decrec-skill](https://github.com/decrec-io/decrec-skill), then add the MCP connector.

**Plugin** ([Claude Code](https://code.claude.com/docs/en/discover-plugins)):

```text
/plugin marketplace add decrec-io/decrec-skill
/plugin install decrec@decrec
```

**MCP connector** ([claude.ai / Desktop / Cowork](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp)): Customize → Connectors → Add custom connector → paste the MCP URL → Connect and sign in.

## ChatGPT

Install the plugin from [decrec-io/decrec-skill](https://github.com/decrec-io/decrec-skill).

In ChatGPT or Codex, open **Plugins**, add that GitHub repo as a marketplace / plugin source, and install **decrec**. Connect when prompted. Start a new chat.

Codex CLI:

```text
codex plugin marketplace add decrec-io/decrec-skill
```

Then install `decrec` from that marketplace. See [ChatGPT plugins](https://learn.chatgpt.com/docs/plugins).

## Cursor

Install the plugin from [decrec-io/decrec-skill](https://github.com/decrec-io/decrec-skill).

In chat:

```text
/add-plugin https://github.com/decrec-io/decrec-skill
```

Or **Customize → Plugins** and install from that GitHub repo. See [Cursor plugins](https://cursor.com/docs/plugins).

## Other

Copy [`skills/decrec/*.md`](https://github.com/decrec-io/decrec-skill/tree/main/skills/decrec) (`SKILL.md` and `explore-prompts.md`) into your client’s skill directory. Add an MCP connector to the MCP URL above.
