# Blink Reminder

**A privacy-first, zero-cost Telegram bot that reminds you to blink and rest your eyes.**

`Fork it` `Set 2 secrets` `Get reminders` `No server needed`

---

## Why?

Staring at screens reduces your blink rate from ~15 to ~5 times per minute, leading to dry eyes, eye strain, and headaches. The **20-20-20 rule** helps: every 20 minutes, look at something 20 feet away for 20 seconds.

This bot sends you periodic Telegram reminders to do exactly that.

## How It Works

```
1. You fork this repo
2. You add your Telegram bot token + chat ID as GitHub Secrets
3. GitHub Actions runs on a cron schedule
4. A curl command sends you a Telegram message
```

No server. No database. No dependencies. Just a GitHub Actions workflow with bash and curl.

## Quick Setup (5 minutes)

1. **Create a Telegram bot** — talk to [@BotFather](https://t.me/BotFather), get a bot token
2. **Get your chat ID** — send a message to your bot, then fetch it from the API
3. **Fork this repository**
4. **Add two GitHub Secrets** to your fork:
   - `TELEGRAM_BOT_TOKEN` — your bot token
   - `TELEGRAM_CHAT_ID` — your chat ID
5. **Enable GitHub Actions** in your fork (Actions tab)
6. **Test it** — go to Actions > Blink Reminder > Run workflow

For detailed step-by-step instructions with screenshots, see **[docs/SETUP.md](docs/SETUP.md)**.

## Privacy & Security

This project is designed to be privacy-first:

- **Your secrets stay in your fork** — tokens are stored as encrypted GitHub Secrets (AES-256)
- **No data collection** — no analytics, no tracking, no external services beyond Telegram
- **No response logging** — the curl command only logs the HTTP status code, never message content
- **You control everything** — it runs in your own GitHub account, you can audit every line

Read the full privacy analysis in **[docs/PRIVACY.md](docs/PRIVACY.md)**.

## Customization

The default schedule sends reminders every 30 minutes during weekday working hours (8am-10pm UTC). You can customize:

- Reminder schedule (different hours, weekends, frequency)
- Message text
- Notification sound (silent mode)
- Rich text formatting

See **[docs/CUSTOMIZATION.md](docs/CUSTOMIZATION.md)** for examples.

## Troubleshooting

Not receiving messages? Workflow not running? See **[docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)**.

## Technical Details

| | |
|---|---|
| **Runtime** | GitHub Actions (free tier: 2,000 min/month for private, unlimited for public) |
| **Language** | Bash + curl |
| **Dependencies** | None |
| **External APIs** | Telegram Bot API |
| **Cost** | $0 |

### Why GitHub Actions?

- Free for public repositories (unlimited minutes)
- No server to maintain or pay for
- Built-in secrets management
- Cron scheduling built in
- Runs in your own account — full control

## Contributing

Contributions are welcome! Please:

1. **Never commit tokens or secrets** — not in code, not in PRs, not in issues
2. Keep the zero-dependency philosophy
3. Maintain privacy-safe logging (no response body logging)

## License

[MIT](LICENSE) — Blink Reminder Contributors
