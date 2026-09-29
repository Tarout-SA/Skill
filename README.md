# Tarout Skills & MCP for Claude Code

Official [Tarout](https://tarout.sa) plugin marketplace. One install gives Claude Code the Tarout skills (deploy playbook + custom domains) and registers the `tarout` MCP server.

## Install

```bash
claude plugin marketplace add Tarout-SA/skills
claude plugin install tarout@tarout
```

Then run `/reload-plugins` (or restart Claude Code) to activate.

The MCP server is the `tarout-mcp` binary shipped with the Tarout CLI — install it once per machine if you haven't:

```bash
curl -fsSL https://tarout.sa/install.sh | sh   # or: npm install -g @tarout/cli
```

It reuses your CLI login (`tarout login`, or any deploy — the CLI opens the browser itself). Headless: `TAROUT_TOKEN` or `tarout login --token <key>`.

## What's inside

| Item | What it does |
|------|--------------|
| `tarout-deploy` skill | The zero-approval deploy playbook: `tarout up --json`, the `needs_input` protocol, fix-and-redeploy on failure, plan upgrades via checkout. |
| `tarout-domains` skill | Custom-domain flow: add-external → DNS records → verify → link, including root domains via Cloudflare-DNS proxied CNAME. |
| `tarout` MCP server | 73 tools (apps, deploys, env vars, scheduled jobs, full database + object-storage control incl. credentials, data browsing, backups, byte transfer, and access keys, domains, billing), including `call`, `list_procedures` and `describe_procedure`, which reach every platform procedure. |

## Other agents (Codex, Cursor, Windsurf, OpenCode, Copilot)

Fetch and follow: **https://tarout.sa/agent-setup/prompt.md** — per-agent setup, validated by Tarout.

Docs for agents: [llms.txt](https://tarout.sa/llms.txt) · [onboarding guide](https://tarout.sa/docs/for-ai/onboarding.md) (every docs page + `.md` serves raw markdown).
