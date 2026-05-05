# Daily Renewal-Window Check

The cron channel fires once on its schedule (9am PT weekdays).
There is no payload to parse — your job is to run the renewal
check and post grouped alerts to Slack.

## What this does

1. Open the vendor sheet (env var `VENDOR_SHEET_ID`, or fall back
   to the most recently-modified sheet titled `Vendors`,
   `Vendor List`, or `Contracts`).
2. For each vendor row, compute `days_until_end` against today
   in `America/Los_Angeles`.
3. Bucket vendors into three groups:
   - **30-day URGENT** (0 ≤ days ≤ 30, plus OVERDUE for past due)
   - **60-day** (31–60)
   - **90-day** (61–90)
4. Compose one Slack message with the URGENT group first, capped
   at 10 vendors per group with `…and N more` overflow.
5. Post to every channel the bot is a member of.

Follow the **Renewal Window Workflow** in SOUL.md (Phases 1–5)
for the exact format, owner-tagging rules, and posting flow.

## Where to post

Resolve target channels per the SOUL **Where to post** section:
list every channel the bot is a member of and post once to each.

## Skip conditions

Skip posting (and stop) if any of these are true:

- **No sheet found AND `VENDOR_SHEET_ID` is unset**: DM the
  workspace install user with a one-line nudge — *"I couldn't
  find a vendor sheet. Set `VENDOR_SHEET_ID` on the agent or
  share me a sheet titled 'Vendors'."* — and stop.
- **No upcoming renewals** (all three groups empty after
  bucketing): stay silent. Post nothing.
- **Bot is in zero channels**: DM the workspace install user
  with the alert digest plus the one-line invite hint from the
  SOUL "Where to post" section.

Post exactly once per resolved destination. Do not retry on
failure — log the error in your session and continue with the
remaining destinations. The next weekday cron fire is the
recovery. Your turn ends after the posts complete.
