# stock-scanner-jobs — unusual-volume pilot

This folder is a reviewed starter for a new **public** repository. It contains no scanner source code and no credential values.

## Security boundary

- The workflow is manual only (`workflow_dispatch`); do not point the external cron service at it yet.
- The public repository's `GITHUB_TOKEN` has read-only access to the public repository.
- Private source is checked out with a separate fine-grained token named `PRIVATE_SCANNER_READ_TOKEN`.
- The private checkout is pinned to reviewed commit `a7f66e059fca0a842a26fd27dc1768e6a5ccc11b`.
- Checkout does not persist the private token in Git configuration.
- No artifacts or caches contain the private checkout.
- Scanner stdout/stderr is redirected to the ephemeral runner and is not uploaded.
- There is deliberately no public dry-run: the current private script prints the complete Discord embed during a dry-run.

## Required GitHub configuration

Create an environment named `unusual-volume-pilot`. During the pilot, protect it with a required reviewer and disable administrator bypass if your plan/settings allow it.

Add these environment secrets:

| Secret | Minimum scope / purpose |
|---|---|
| `PRIVATE_SCANNER_READ_TOKEN` | Fine-grained PAT selected only for `kfirAdler/stock-scanner`, repository permission **Contents: Read-only**, no other repository permissions |
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL used by the scanner |
| `SUPABASE_SERVICE_ROLE_KEY` | Existing service-role credential; consider replacing it later with a narrower database/RPC credential |
| `DISCORD_BOT_TOKEN` | Existing Discord bot credential |
| `DISCORD_UNUSUAL_VOLUME_CHANNEL_ID` | Destination channel identifier (kept as a secret to preserve the current boundary) |

Do not use a classic PAT. Give the fine-grained token an expiry date and rotate it. A read-only deploy key scoped to this one private repository is an even narrower future alternative, but this pilot follows the requested token design.

## Pilot sequence

1. Create the public repository with Actions enabled and push only this folder's contents.
2. Configure the protected environment and its secrets.
3. Run **Unusual volume pilot** with `mode=validate`. This does not query market data, mutate Supabase, or send to Discord.
4. Inspect the entire public log. It should show only checkout/setup metadata and the generic validation message. Verify that no private source, credential, URL, embed, ticker list, or traceback appears.
5. Run once with `mode=send` while watching the intended Discord channel and Supabase delivery table. Application output remains suppressed.
6. Confirm deduplication by checking that the expected delivery rows/messages were created and that no duplicate message was sent.
7. Only after those checks, repoint the external cron service from the private workflow to this public workflow. Keep the old private workflow disabled until at least several successful scheduled runs have completed.

## Rollback

Disable Actions in the public repository or remove its environment secrets, then point the cron service back to the private repository. The private workflow should remain present and disabled during the observation period.
