# Customization

All customization is done by editing `.github/workflows/blink-reminder.yml` in your fork.

## Change the Schedule

The schedule is controlled by the `cron` field. The default is:

```yaml
- cron: "*/30 8-22 * * 1-5"
```

This means: every 30 minutes, hours 8-22 UTC, Monday through Friday.

### Cron Format

```
┌───────────── minute (0-59)
│ ┌───────────── hour (0-23)
│ │ ┌───────────── day of month (1-31)
│ │ │ ┌───────────── month (1-12)
│ │ │ │ ┌───────────── day of week (0-6, Sunday=0)
│ │ │ │ │
* * * * *
```

### Schedule Examples

| Schedule | Cron Expression | Description |
|----------|----------------|-------------|
| Every 30 min, work hours | `*/30 8-22 * * 1-5` | Default |
| Every hour, work hours | `0 8-22 * * 1-5` | Less frequent |
| Every 20 min, work hours | `*/20 8-22 * * 1-5` | More frequent |
| Every day including weekends | `*/30 8-22 * * *` | 7 days a week |
| Only mornings | `*/30 8-12 * * 1-5` | 8am-12pm UTC |
| Once at start of each hour | `0 8-22 * * 1-5` | On the hour |

> **Tip:** Use [crontab.guru](https://crontab.guru/) to build and validate cron expressions.

> **Note:** GitHub Actions cron schedules may have delays of a few minutes. It's not a precision timer.

## Timezone Adjustments

GitHub Actions cron uses **UTC**. Convert your local time to UTC:

| Your Timezone | UTC Offset | 9am-6pm Local = UTC |
|---------------|-----------|---------------------|
| US Eastern (EST) | UTC-5 | `14-23` |
| US Eastern (EDT) | UTC-4 | `13-22` |
| US Pacific (PST) | UTC-8 | `17-2` (next day) |
| US Pacific (PDT) | UTC-7 | `16-1` (next day) |
| Central Europe (CET) | UTC+1 | `8-17` |
| Central Europe (CEST) | UTC+2 | `7-16` |
| India (IST) | UTC+5:30 | `3-12` |
| Japan/Korea (JST/KST) | UTC+9 | `0-9` |

For example, if you're in US Eastern (EST) and want reminders 9am-6pm local:

```yaml
- cron: "*/30 14-23 * * 1-5"
```

> **Note:** You may need to update the cron when daylight saving time changes.

## Change the Message

Find the `MESSAGE=` line in the workflow and replace it:

```yaml
- name: Send blink reminder
  run: |
    MESSAGE="Your custom message here"
    # ... rest stays the same
```

### Rich Text (Markdown)

Telegram supports Markdown formatting. Add `\"parse_mode\": \"Markdown\"` to the JSON payload:

```yaml
- name: Send blink reminder
  run: |
    MESSAGE="*Blink Reminder* 👀\n\nLook away from the screen for 20 seconds.\n_Your eyes will thank you._"

    HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
      -X POST "https://api.telegram.org/bot${{ secrets.TELEGRAM_BOT_TOKEN }}/sendMessage" \
      -H "Content-Type: application/json" \
      -d "{\"chat_id\": \"${{ secrets.TELEGRAM_CHAT_ID }}\", \"text\": \"${MESSAGE}\", \"parse_mode\": \"Markdown\"}")

    if [ "$HTTP_STATUS" -eq 200 ]; then
      echo "Reminder sent successfully (HTTP $HTTP_STATUS)"
    else
      echo "::error::Failed to send reminder (HTTP $HTTP_STATUS)"
      exit 1
    fi
```

## Silent Notifications

To send reminders without a notification sound, add `\"disable_notification\": true`:

```yaml
-d "{\"chat_id\": \"${{ secrets.TELEGRAM_CHAT_ID }}\", \"text\": \"${MESSAGE}\", \"disable_notification\": true}"
```

This is useful if you want a gentle reminder that doesn't interrupt your flow — you'll see it next time you check Telegram.

## Multiple Reminder Types

You can add multiple cron triggers or duplicate the job with different messages. For example, a short reminder every 30 min and a longer break reminder every 2 hours:

```yaml
on:
  schedule:
    - cron: "*/30 8-22 * * 1-5"   # Every 30 min
    - cron: "0 10,12,14,16,18,20 * * 1-5"  # Every 2 hours

jobs:
  send-reminder:
    runs-on: ubuntu-latest
    steps:
      - name: Validate secrets
        run: |
          if [ -z "${{ secrets.TELEGRAM_BOT_TOKEN }}" ]; then
            echo "::error::TELEGRAM_BOT_TOKEN secret is not set."
            exit 1
          fi
          if [ -z "${{ secrets.TELEGRAM_CHAT_ID }}" ]; then
            echo "::error::TELEGRAM_CHAT_ID secret is not set."
            exit 1
          fi

      - name: Send blink reminder
        run: |
          CRON="${{ github.event.schedule }}"

          if [ "$CRON" = "0 10,12,14,16,18,20 * * 1-5" ]; then
            MESSAGE="🧘 Time for a longer break! Stand up, stretch, and look out the window for a minute."
          else
            MESSAGE="👀 Time to blink! Look away from the screen for 20 seconds. Your eyes will thank you."
          fi

          HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
            -X POST "https://api.telegram.org/bot${{ secrets.TELEGRAM_BOT_TOKEN }}/sendMessage" \
            -H "Content-Type: application/json" \
            -d "{\"chat_id\": \"${{ secrets.TELEGRAM_CHAT_ID }}\", \"text\": \"${MESSAGE}\"}")

          if [ "$HTTP_STATUS" -eq 200 ]; then
            echo "Reminder sent successfully (HTTP $HTTP_STATUS)"
          else
            echo "::error::Failed to send reminder (HTTP $HTTP_STATUS)"
            exit 1
          fi
```

> **Note:** `${{ github.event.schedule }}` contains the cron expression that triggered the run, so you can use it to differentiate between schedules.
