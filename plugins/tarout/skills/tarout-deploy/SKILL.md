---
name: tarout-deploy
description: Deploy applications to Tarout (tarout.sa), the Saudi cloud platform. Use when the user wants to deploy, ship, host, or publish an app to Tarout or to Saudi cloud infrastructure, mentions tarout.sa or the tarout CLI, or needs Tarout operations — apps, deployments, logs, environment variables, managed Postgres/MySQL databases, S3-compatible storage, or billing. Covers the full zero-approval deploy flow, JSON agent mode, failure recovery, and paid-plan upgrades.
---

# Deploy to Tarout

Tarout is a PaaS (Vercel/Heroku-like) on Saudi infrastructure — region `me-central2` (Dammam). Apps, managed Postgres/MySQL, S3-compatible storage, custom domains, AI gateway. Everything is driven by the `tarout` CLI; every platform capability is also reachable via `tarout call <router.procedure>`.

## The contract: deploy end-to-end, never wait for approval

The user asking you to deploy IS the approval. Announce what you are about to do in one short message, then run it in the same turn — no "shall I proceed?", no plan-then-wait. Complete 100% of the deployment; **the deliverable is the live application URL**. The only steps the human performs are the ones Tarout opens in the browser: login/registration and payment. Ask permission only before destructive actions (deleting resources, cancelling/rolling back deployments, overwriting another app's source, revealing existing secret values, leaving the Saudi region).

## Setup (once per machine / project)

```bash
# Install if missing (installs `tarout` + `tarout-mcp`):
curl -fsSL https://tarout.sa/install.sh | sh    # or: npm install -g @tarout/cli

# Grant hands-free tarout permissions in this project (writes CLAUDE.md + allowlist):
tarout agent init --agent claude
```

The CLI self-updates on `up`/`deploy` when a newer version is published (opt out: `--no-update-check`).

## Deploy

Inspect the project first (database usage: `prisma/schema.prisma`, `pg`/`mysql2`/`drizzle-orm` deps, `DATABASE_URL`; storage usage: S3/GCS/UploadThing/multer deps). Then announce the plan as a statement and run:

```bash
tarout up --json --yes --new-app        # new app; use --app <id|name> to redeploy an existing one
```

`tarout up` inspects, handles auth (opens browser login/registration itself — never ask the user to log in first), creates the app on the org's subscribed tier (never pass `--plan free`), uploads the current directory, deploys, and streams JSON events. Read the final line's envelope: `success` and `data.url` (or `error.code`). Add `--database postgres|mysql` / `--storage` when the code needs them — announce, don't ask; resources land on the subscribed tier automatically.

Parse stdout line by line. A `{"type":"needs_input", ...}` line (exit code 6) means a flag is missing: **answer it yourself** when derivable (name → folder name, create-vs-reuse → `--new-app`, source → `--source upload`, region → omit, confirms → `--yes`), re-run with the event's `flag` + answer. Relay to the human only what you cannot know: an API token on a headless machine (`tarout login --token <key>` / `TAROUT_TOKEN`) or a payment.

Exit codes: 0 success · 1 general · 2 invalid args · 3 auth · 4 not found · 5 permission denied · 6 needs input · 10 deployment failed · 11 deployment timeout · 12 build failed · 13 checkout pending.

## When a deploy fails: fix and redeploy

Never stop at the first failure and never retry unchanged. Read `error.details.errorAnalysis.suggestedFixes` and `tarout deploy:logs <deployment-id>`, fix the project yourself — missing start script, app not listening on `PORT`, missing lockfile/dependency, missing env var (`tarout env <app> set KEY=value`), oversized upload (prune `node_modules`/build output) — then redeploy. Up to 3 fix attempts; stop only when the same error survives a fix with nothing left to change, or the fix needs something only the user has.

## Paid plans (`NEEDS_UPGRADE` / plan limit reached)

Don't pre-ask in chat. Tell the user you're opening Tarout checkout and run:

```bash
tarout billing upgrade --json --yes --plan <planKey> --billing-period monthly
```

It opens the payment page and polls to completion — completing payment IS the user's approval; closing checkout is refusal. Then retry the blocked command.

## Env vars & secrets

Setting the variables the app needs to run (generated auth secrets, `DATABASE_URL` wiring) is part of the deploy — do it and say so (`tarout env <app> set KEY=value`). Ask only before revealing existing secret values (`tarout env <app> reveal KEY`).

## Report the result

End with a compact block: app name + live URL (`data.url`), database (type or none), storage, region `me-central2 (Dammam)`, dashboard link `https://tarout.sa/dashboard`. Offer next steps: env vars, custom domain (see the tarout-domains skill), `tarout dev` for local dev with cloud env vars.

## Full references (raw markdown, agent-friendly)

- Complete onboarding guide: https://tarout.sa/docs/for-ai/onboarding.md
- CLI reference: https://tarout.sa/docs/for-ai/cli-reference.md
- JSON envelope schema: https://tarout.sa/docs/for-ai/cli-json-schema.md
- Everything else: https://tarout.sa/llms.txt (any docs page + `.md` serves raw markdown)
