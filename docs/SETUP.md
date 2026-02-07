# Setup Guide

Get your blink reminders running in about 5 minutes.

## Prerequisites

- A GitHub account
- A Telegram account
- A phone or computer with Telegram installed

## Step 1: Create a Telegram Bot

1. Open Telegram and search for **@BotFather**
2. Send the command `/newbot`
3. Choose a name for your bot (e.g., "My Blink Reminder")
4. Choose a username for your bot (must end in `bot`, e.g., `my_blink_reminder_bot`)
5. BotFather will reply with your **bot token** — it looks like `123456789:ABCdefGhIjKlMnOpQrStUvWxYz`
6. **Save this token** — you'll need it in Step 4

> **Important:** Keep your bot token secret. Anyone with the token can control your bot.

## Step 2: Get Your Chat ID

1. Open a chat with your new bot in Telegram
2. Send it any message (e.g., "hello")
3. Open this URL in your browser (replace `YOUR_BOT_TOKEN` with the token from Step 1):

   ```
   https://api.telegram.org/botYOUR_BOT_TOKEN/getUpdates
   ```

4. Look for `"chat":{"id":` in the JSON response — that number is your **chat ID**
5. It will look something like `123456789` (a positive number for personal chats)

> **Tip:** If you see an empty `"result":[]`, make sure you sent a message to the bot first, then refresh the URL.

### Example Response

```json
{
  "ok": true,
  "result": [
    {
      "message": {
        "chat": {
          "id": 123456789,
          "type": "private"
        },
        "text": "hello"
      }
    }
  ]
}
```

Your chat ID in this example is `123456789`.

## Step 3: Fork the Repository

1. Go to this repository on GitHub
2. Click the **Fork** button (top right)
3. Keep the default settings and click **Create fork**

## Step 4: Add GitHub Secrets

In your **forked** repository:

1. Go to **Settings** > **Secrets and variables** > **Actions**
2. Click **New repository secret**
3. Add the first secret:
   - **Name:** `TELEGRAM_BOT_TOKEN`
   - **Secret:** Your bot token from Step 1
4. Click **Add secret**
5. Click **New repository secret** again
6. Add the second secret:
   - **Name:** `TELEGRAM_CHAT_ID`
   - **Secret:** Your chat ID from Step 2
7. Click **Add secret**

## Step 5: Enable GitHub Actions

GitHub may disable Actions on forked repositories by default:

1. Go to the **Actions** tab in your fork
2. If you see a prompt to enable workflows, click **I understand my workflows, go ahead and enable them**

## Step 6: Test It

1. Go to the **Actions** tab
2. Click **Blink Reminder** in the left sidebar
3. Click **Run workflow** > **Run workflow**
4. Wait for the workflow to complete (should take a few seconds)
5. Check your Telegram — you should have received a reminder!

If the workflow succeeded but you didn't get a message, see [TROUBLESHOOTING.md](TROUBLESHOOTING.md).

## What Happens Next

With the default settings, you'll receive a reminder:
- Every 30 minutes
- Between 8:00 AM and 10:00 PM **UTC**
- Monday through Friday

To adjust the schedule, timezone, or message, see [CUSTOMIZATION.md](CUSTOMIZATION.md).

## Setup Troubleshooting

### "TELEGRAM_BOT_TOKEN secret is not set"
You haven't added the `TELEGRAM_BOT_TOKEN` secret yet, or there's a typo in the secret name. Go to Settings > Secrets and variables > Actions and verify.

### "TELEGRAM_CHAT_ID secret is not set"
Same as above but for `TELEGRAM_CHAT_ID`.

### HTTP 401 Error
Your bot token is invalid or expired. Create a new bot with @BotFather and update the secret.

### HTTP 400 Error
Your chat ID is likely incorrect. Re-do Step 2 to get the correct ID.

### Empty getUpdates response
You need to send a message to your bot in Telegram **before** calling the getUpdates API. Send any message, then try the URL again.
