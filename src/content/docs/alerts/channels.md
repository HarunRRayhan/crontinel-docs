---
title: Alert Channels
description: Configuring Slack, email, PagerDuty, and webhook alerts
---

## Available channels

| Channel | Config key | Notes |
|---|---|---|
| Slack | `slack.webhook_url` | Incoming webhook URL |
| Email | `mail.to` | Recipient email address |
| PagerDuty | — | Coming soon |
| Webhook | `webhook.url` | Any HTTPS endpoint |

## Configuration

```env
# Slack
CRONTINEL_ALERT_CHANNEL=slack
CRONTINEL_SLACK_WEBHOOK=https://hooks.slack.com/services/T.../B.../...

# Email
CRONTINEL_ALERT_CHANNEL=mail
CRONTINEL_ALERT_EMAIL=ops@yourcompany.com
```

## Alert deduplication

The same condition won't re-fire for 5 minutes. If a queue stays above its threshold for an hour, you get one alert at the start — not one every poll cycle.

## Auto-resolution

When a condition clears, Crontinel sends a "resolved" notification. You get clear start and end signals for every incident.

## SaaS alert management

In the [Crontinel SaaS](https://app.crontinel.com) you can configure per-app alert channels through the web UI without touching config files.
