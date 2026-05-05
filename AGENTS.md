This folder contains the source for a Skilled Agent originally built for the Valet runtime. Changes should follow the Skilled Agent open standard.

## Setup

### Connectors

- **google-sheets**: The Google Sheets MCP server, OAuth-authenticated. The agent uses it to read the vendor list spreadsheet — vendor name, owner, owner email, approval status, contract end date — and (when explicitly asked in Slack) to update cells or append rows. Add it from the catalog at the org level so other Sheets-powered agents can share it.

### Channels

- **slack** (slack): The agent's per-agent Slack bot. Listens for @mentions and replies in-thread, and posts the daily renewal digest to whichever channels the bot has been invited to. Slack writes use the auto-injected outbound Slack connector.
- **cron** (cron): Fires the daily vendor renewal-window check at 9am Pacific, Monday through Friday (`0 9 * * 1-5`, `America/Los_Angeles`). Declared inline in `valet.yaml`, so it's created automatically by the dashboard setup flow.

### Secrets

This agent uses the OAuth variant of Google Sheets, so no API token is needed at the org or agent level. The OAuth grant happens in the dashboard setup flow when you connect Google.

### External Setup

1. After deploy, OAuth into Google so the agent can read your vendor sheet.
2. Set the `VENDOR_SHEET_ID` env var on the agent to the Google Sheet ID of your vendor list (the long id from the sheet URL: `docs.google.com/spreadsheets/d/<VENDOR_SHEET_ID>/edit`). The agent will pick it up on the next cron fire. If unset, the agent falls back to the most recently-modified Google Sheet whose title looks like a vendor list (`Vendors`, `Vendor List`, `Contracts`); set the env var explicitly to remove the guesswork.
3. Invite the agent's Slack bot to your procurement channel (or wherever your team handles vendor renewals). The agent posts to every channel it's a member of — invite it to one focused channel, or several. If the bot has not been invited anywhere, the digest is sent as a DM to the workspace install user with a one-line nudge.
4. Invite the bot to any additional channels where teammates should be able to @mention it for ad-hoc vendor questions.
5. The first cron fire is the next 9am Pacific weekday after deploy. To smoke-test sooner, @mention the bot with a question like *"which contracts are renewing in 60 days?"* — that exercises the Slack + Sheets path without waiting for the cron.

## Customizing

- **Change the schedule**: edit the `cron` and `timezone` on the `cron` channel in `valet.yaml`, then redeploy. The default is `0 9 * * 1-5` `America/Los_Angeles` (9am PT weekdays).
- **Change the alert windows**: the SOUL.md "Renewal Window Workflow" defines three buckets — 30-day URGENT, 60-day, 90-day. Edit those thresholds (and the URGENT prefix) in SOUL.md to match how far ahead your procurement team needs to act.
- **Pin to a specific sheet**: set `VENDOR_SHEET_ID` on the agent. With it set, the agent will not search by title — it always reads the sheet you pinned.
- **Control where alerts post**: invite or remove the bot from channels in Slack — that's the only signal the agent uses. There is no channel name in the configuration.
