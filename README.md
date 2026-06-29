# ai-agents

A vendor-neutral home for **Agent Skills** — portable skill packages that follow the open
[Agent Skills](https://agentskills.io) standard. Each skill is a folder containing a `SKILL.md`
file (with `name` + `description` metadata) plus optional `references/`, `scripts/`, and
`assets/`. Any compatible agent can load them: Claude, OpenCode, Cursor, OpenAI Codex,
Gemini CLI, and many others.

## Skills

| Skill | Description |
|-------|-------------|
| [`serpzilla-seo-guest-posting`](./serpzilla-seo-guest-posting) | Buy SEO guest posts and link insertions on trusted donor sites via the Serpzilla platform. |

---

## Serpzilla SEO Guest Posting

This skill enables an agent to automate backlink acquisition through
[Serpzilla](https://serpzilla.com): create projects, search and filter donor sites by
DR / traffic / price / language, upload articles, purchase placements, and manage the
placement lifecycle. It requires the **Serpzilla MCP server** to be connected to the agent.

### Features

- Create projects for promoted websites
- Search and filter donor sites (price, DR, traffic, language, Majestic/Moz/Ahrefs/Semrush metrics, …)
- Upload articles for moderation (`news` format)
- Buy four placement types: `news`, `review`, `link`, `archive`
- Manage purchased placements (approve, cancel, terminate, …) with built-in financial guardrails

### 1. Connect the Serpzilla MCP

The Serpzilla MCP is a hosted **streamable HTTP** server. Connect it using only the URL —
authentication is handled by a built-in OAuth flow, so no API token needs to be pasted into
the agent config.

```
https://mcp.serpzilla.com/mcp
```

How you add it depends on your agent. For example, in **Claude Desktop** open
*Settings → Developer → Edit Config* and add it to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "serpzilla": {
      "type": "http",
      "url": "https://mcp.serpzilla.com/mcp"
    }
  }
}
```

> On older clients that predate streamable HTTP, use `"type": "sse"` instead of `"http"`.
> On claude.ai, add it under *Settings → Connectors* using the same URL.

### 2. Install the skill

The skill is the `serpzilla-seo-guest-posting/` folder. Package it so that the **folder is the
root** of the archive (per the Agent Skills spec), then load it into your agent:

```bash
zip -r serpzilla-seo-guest-posting.zip serpzilla-seo-guest-posting
```

Then, in Claude for example, go to *Customize → Skills* and upload the archive. Other agents
that follow the standard accept the same package; see the [Agent Skills clients](https://agentskills.io)
list.

### 3. Use it

Ask the agent in natural language, e.g. *"Find donor sites with DR 30+ for my site
example.com and buy a guest post under $50."* The skill guides the agent through the workflow
and enforces confirmation before any purchase or money-moving action.

You still need a Serpzilla **advertiser** account; get an API token in
[Passport → Security](https://passport.serpzilla.com/security/token/). Top up your balance at
<https://passport.serpzilla.com/deposit/>.

## Repository contents

```
ai-agents/
├── README.md                              # this file (repo-level, not part of any skill package)
├── .gitignore
└── serpzilla-seo-guest-posting/           # the skill package
    ├── SKILL.md                           # metadata + workflow instructions
    └── references/
        └── placement_actions.md           # placement actions & statuses reference
```

## See also

- [Agent Skills standard](https://agentskills.io) — the open format these skills follow
- [How to create custom skills (Claude Help Center)](https://support.claude.com/en/articles/12512198-how-to-create-custom-skills)
- [Serpzilla API documentation](https://serpzilla.com/api/)
- [MCP server source (Docker/stdio variant)](https://github.com/stanislav-reshetnev/serpzilla-mcp-server)
