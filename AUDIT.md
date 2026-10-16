# Existing Actions audit (2026-10-16)

Source inspected: `kfirAdler/stock-scanner`, remote `origin/main` at `a7f66e059fca0a842a26fd27dc1768e6a5ccc11b`.

## Current workflow inventory

| Workflow | Trigger currently present | Explicitly disabled | Secrets referenced |
|---|---|---:|---|
| `unusual-volume.yml` | manual dispatch | yes | `NEXT_PUBLIC_SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `DISCORD_BOT_TOKEN`, `DISCORD_UNUSUAL_VOLUME_CHANNEL_ID` |
| `refresh-market-snapshot.yml` | manual dispatch | yes | Supabase URL/service role, Discord bot token |
| `pattern-breakouts.yml` | manual dispatch | yes | Supabase URL/service role, Discord bot token |
| `daily-market-news.yml` | manual dispatch | yes | Supabase URL/service role, Discord bot/token/channel/webhook values |
| `economic-calendar.yml` | manual dispatch | yes | Discord bot token and channel ID |
| `fear-greed.yml` | manual dispatch | yes | Discord bot token and channel ID |
| `live-stock-news.yml` | manual dispatch | yes | Supabase URL/service role, Discord values, Finnhub API key |
| `market-rss-headlines.yml` | manual dispatch | yes | Discord bot/channel/webhook values |

Repository variables are also referenced by some non-pilot workflows for channel IDs. Two pattern channel IDs are currently hard-coded in the private refresh workflow; they are identifiers rather than authentication credentials, but they should not be copied blindly into the public repository.

## Unusual-volume findings

- Current permissions are `contents: read`; this is appropriate for its own repository but cannot read a different private repository.
- Concurrency cancels the prior in-progress run, which is suitable for a frequent scanner.
- Runtime limit is 15 minutes.
- `send_to_discord` defaults to false.
- The job is currently disabled by `if: false`.
- The existing dry-run prints the generated Discord embed to stdout. In a public repository that output becomes public, so the pilot does not expose a dry-run mode.
- A send run writes deduplication state to Supabase and sends to Discord only when a new RVOL milestone is found. The script prints a Discord message ID after a send; the pilot suppresses all application output.
- The script loads a local `.env.local` if present, but that file is ignored by Git and is not included in the public pilot.

## Trigger and cron migration

The inspected workflows contain only `workflow_dispatch`; no `repository_dispatch` trigger is present. The external cron service therefore needs credentials capable of dispatching the public workflow. Keep that cron credential separate from `PRIVATE_SCANNER_READ_TOKEN`; it does not belong in the repository and must not gain access to the private scanner repository.

Do not change the cron target until both manual pilot modes have been reviewed. When migrating, preserve the current schedule in the external service, change only the repository/workflow target, and retain a quick rollback path.
