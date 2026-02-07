# Privacy & Security

This project is designed with privacy as a core principle. This document explains exactly what data flows where, what's protected, and what risks remain.

## Data Flow

```
┌──────────────────┐    cron trigger     ┌──────────────────┐
│  GitHub Actions   │ ◄──────────────── │  GitHub Scheduler  │
│  (your fork)      │                    └──────────────────┘
│                   │
│  Reads secrets:   │
│  - BOT_TOKEN      │
│  - CHAT_ID        │
│                   │
│  Runs curl:       │
│  POST /sendMessage│ ──────────────────► ┌──────────────────┐
│                   │   HTTPS (TLS 1.2+)  │  Telegram API     │
│  Logs only:       │                     │                   │
│  HTTP status code │ ◄────────────────── │  Delivers message │
└──────────────────┘   200 / 4xx / 5xx    │  to your chat     │
                                          └──────────────────┘
```

## What's Stored Where

| Data | Where | Encryption | Who Can Access |
|------|-------|-----------|----------------|
| Bot token | GitHub Secrets | AES-256, encrypted at rest | Only you (repo owner) |
| Chat ID | GitHub Secrets | AES-256, encrypted at rest | Only you (repo owner) |
| Workflow file | Your fork repo | N/A (public if repo is public) | Anyone (if public) / you (if private) |
| Message content | Hardcoded in workflow | N/A | Anyone (if repo is public) |
| Telegram messages | Telegram servers | Telegram's encryption | You and Telegram |
| Workflow logs | GitHub Actions | GitHub's encryption | Anyone (if public) / you (if private) |

## What's NOT Stored

- No database
- No external analytics or tracking
- No cookies or browser storage
- No third-party services beyond Telegram
- No logs of message content (curl output is suppressed)

## How GitHub Secrets Work

GitHub Secrets are:
- Encrypted with AES-256 before they reach GitHub's servers
- Never exposed in workflow logs (GitHub automatically masks them)
- Not available to forks' pull requests (prevents secret theft via PRs)
- Accessible only to the repository owner and workflows in that repository
- Not included in repository exports or API responses

Read more: [GitHub Docs — Encrypted Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

## Threat Model

### Protected Against

- **Token exposure in logs** — curl uses `-s -o /dev/null -w "%{http_code}"` to suppress all output except the HTTP status code
- **Token exposure in code** — tokens are stored as GitHub Secrets, never hardcoded
- **Secret theft via PRs** — GitHub does not expose secrets to workflows triggered by pull requests from forks
- **Transit interception** — Telegram API is accessed over HTTPS (TLS 1.2+)

### Not Protected Against

- **GitHub compromise** — if GitHub itself is breached, secrets could theoretically be exposed (extremely unlikely, same risk as any GitHub-hosted project)
- **Telegram-side access** — Telegram can read messages sent through their API (same as any Telegram message)
- **Workflow file visibility** — if your fork is public, anyone can see your cron schedule and message text (but not your secrets)
- **Your own device** — if someone has access to your GitHub account, they can read/modify secrets

### Risk Summary

The practical risk is very low. Your bot token and chat ID are encrypted at rest, masked in logs, and transmitted over HTTPS. The only realistic attack vector is someone gaining access to your GitHub account.

## Best Practices

1. **Enable 2FA on GitHub** — protects your secrets if your password is compromised
2. **Enable 2FA on Telegram** — protects your Telegram account
3. **Use a private fork** if you want to hide your schedule and message text
4. **Rotate your bot token** periodically — revoke the old one via @BotFather's `/revokentoken` command and update the GitHub Secret
5. **Review workflow runs** occasionally to confirm expected behavior

## How to Audit

This project is fully auditable:

1. **Read the workflow file** — it's the only executable code, and it's short
2. **Check workflow logs** — verify only HTTP status codes are logged
3. **Inspect the curl command** — `-s -o /dev/null -w "%{http_code}"` ensures no response body is captured
4. **Review GitHub Secrets settings** — confirm only two secrets exist
5. **Search the entire repo** — no other files contain executable code

## GDPR Notes

- **Data processed:** Telegram chat ID (pseudonymous identifier), bot token (credential)
- **Data controller:** You (the person who forks and configures the repository)
- **Data processor:** GitHub (runs the workflow), Telegram (delivers messages)
- **Legal basis:** Consent (you set it up for yourself)
- **Data deletion:** Delete the fork and revoke the bot token via @BotFather to remove all data

Since you are both the data controller and the data subject, GDPR compliance is straightforward — you have full control over your own data at all times.

## Reporting Security Issues

If you discover a security vulnerability in this project, please open an issue on the repository. Since the project contains no server-side code and secrets are per-user, most security concerns will relate to workflow best practices.

For issues with GitHub Secrets or GitHub Actions security, report to [GitHub Security](https://github.com/security).

For issues with the Telegram Bot API, report to [Telegram](https://telegram.org/support).
