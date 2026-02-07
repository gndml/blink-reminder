# Troubleshooting

## Not Receiving Messages

### 1. Check the workflow status

Go to the **Actions** tab in your fork. Look for the most recent **Blink Reminder** run:

- **Green checkmark** — workflow ran successfully. If you still didn't get a message, the issue is on Telegram's side.
- **Red X** — workflow failed. Click on the run to see the error message.
- **No runs at all** — the workflow hasn't been triggered yet. See "Workflow Not Running" below.

### 2. Did you message the bot first?

Telegram bots can only send messages to users who have started a conversation with the bot. Open your bot in Telegram and send it any message (e.g., "hello"), then try again.

### 3. Check your secrets

Go to **Settings** > **Secrets and variables** > **Actions** in your fork. Verify:

- `TELEGRAM_BOT_TOKEN` exists (you can't see the value, but you can see the name)
- `TELEGRAM_CHAT_ID` exists
- There are no typos in the secret names (they're case-sensitive)

### 4. Check the HTTP status code

In the workflow run logs, look for the HTTP status code:

| Status | Meaning | Fix |
|--------|---------|-----|
| 200 | Success | Message was sent — check Telegram |
| 400 | Bad request | Chat ID is likely incorrect. Re-fetch it (see [SETUP.md](SETUP.md#step-2-get-your-chat-id)) |
| 401 | Unauthorized | Bot token is invalid. Create a new bot or revoke/regenerate the token via @BotFather |
| 403 | Forbidden | Bot was blocked by the user, or chat ID is wrong |
| 404 | Not found | Bot token format is wrong (missing `bot` prefix is handled by the URL, so check for extra characters) |
| 429 | Rate limited | Too many messages sent. Wait and try again. Consider reducing frequency |
| 500+ | Server error | Telegram is having issues. Wait and try again |

### 5. Test manually

Trigger the workflow manually: **Actions** > **Blink Reminder** > **Run workflow** > **Run workflow**. This rules out cron scheduling issues.

## Workflow Not Running

### Inactive repository auto-disable

GitHub automatically disables scheduled workflows in repositories with no activity for **60 days**. To fix:

1. Make any commit to the repository (even editing the README)
2. Or go to **Actions** and re-enable the workflow

### Actions not enabled

Forked repositories may have Actions disabled by default:

1. Go to the **Actions** tab
2. If prompted, click **I understand my workflows, go ahead and enable them**

### Cron syntax error

If the cron expression is invalid, the workflow won't be scheduled. Validate your expression at [crontab.guru](https://crontab.guru/).

Common mistakes:
- Using `7` for Sunday (should be `0` or `7` — both work, but `0` is standard)
- Using `24` for midnight (should be `0`)
- Forgetting that hours are 0-23, not 1-24

## Messages Delayed

GitHub Actions cron schedules are **not exact**. GitHub may delay execution by several minutes, especially during high-demand periods. This is normal and expected.

If you need precise timing, GitHub Actions is not the right tool — consider a VPS with a real cron job instead. For blink reminders, a few minutes of variance doesn't matter.

## Bot Token Invalid

If your token has been compromised or you need to rotate it:

1. Open Telegram and talk to **@BotFather**
2. Send `/mybots`
3. Select your bot
4. Select **API Token** > **Revoke current token**
5. Copy the new token
6. Go to your fork: **Settings** > **Secrets and variables** > **Actions**
7. Click the pencil icon next to `TELEGRAM_BOT_TOKEN`
8. Paste the new token and save

## Chat ID Invalid

To re-fetch your chat ID:

1. Send a new message to your bot in Telegram
2. Open: `https://api.telegram.org/botYOUR_BOT_TOKEN/getUpdates`
3. Find `"chat":{"id":` in the response
4. Update the `TELEGRAM_CHAT_ID` secret in your fork

## Debugging Checklist

Run through this list in order:

- [ ] Is the workflow enabled? (Actions tab shows the workflow)
- [ ] Are both secrets set? (`TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID`)
- [ ] Did you message the bot in Telegram first?
- [ ] Does a manual trigger (Run workflow) work?
- [ ] Is the HTTP status code 200?
- [ ] Has the repo had activity in the last 60 days? (prevents auto-disable)
- [ ] Is the cron expression valid? (test at [crontab.guru](https://crontab.guru/))
- [ ] Are the secret names exactly right? (case-sensitive, no extra spaces)

If you've gone through everything and it still doesn't work, open an issue on the repository with:
- The HTTP status code from the workflow logs (do **not** share your bot token or chat ID)
- Which step in the checklist above fails
