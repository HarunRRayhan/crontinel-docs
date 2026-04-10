---
title: Status Pages
description: Give your users a public status page showing real-time uptime, incident history, and per-monitor health – included free with every Crontinel account.
---

## Why a status page

When something breaks, your users want answers before you've even opened your laptop. A public status page gives them a single URL to check instead of flooding your inbox. Crontinel includes one free status page with every account – no separate product, no extra setup.

## Creating your first status page

Your status page is auto-created after your first monitor reports a successful check. You can also create one manually from the dashboard under **Status Pages → Create**.

Each page gets a public URL:

```
https://crontinel.com/status/your-app
```

The slug is editable. Pick something short and recognizable – your users will bookmark it.

## Adding monitors

From **Status Pages → Edit**, select which monitors appear on the page. Each monitor shows as a row with:

- Current status indicator
- 90-day history bar (green/yellow/red dots)
- Uptime percentage

Only the monitors you choose are visible. Internal or sensitive checks stay private.

## Sharing your URL

Three URL formats depending on your plan:

| Tier | Format |
|------|--------|
| Free | `crontinel.com/status/{slug}` |
| Pro | `{slug}.status.crontinel.com` |
| Team | Your own custom domain |

Pages are server-rendered HTML – no JavaScript required, fully indexable by search engines.

## Embeddable badges

Every status page includes an SVG badge you can embed anywhere.

**HTML:**

```html
<a href="https://crontinel.com/status/your-app">
  <img src="https://crontinel.com/badge/your-app.svg" alt="Status">
</a>
```

**Markdown (GitHub README):**

```markdown
[![Status](https://crontinel.com/badge/your-app.svg)](https://crontinel.com/status/your-app)
```

The badge updates automatically and shows current overall status.

## Customization

| Feature | Free | Pro | Team |
|---------|------|-----|------|
| Title and description | Yes | Yes | Yes |
| Custom logo | – | Yes | Yes |
| Custom colors | – | Yes | Yes |
| Remove Crontinel branding | – | Yes | Yes |
| Subdomain URL | – | Yes | Yes |
| Custom domain | – | – | Yes |

Free pages display a small "Monitored by Crontinel" footer badge.

## Incident management

Post incidents manually from **Status Pages → Incidents → New Incident**. Each incident moves through four stages:

1. **Investigating** – you're aware and looking into it
2. **Identified** – root cause found
3. **Monitoring** – fix deployed, watching for recurrence
4. **Resolved** – back to normal

Updates appear on the status page in reverse chronological order. Your users see exactly what you post – nothing is auto-generated.

## Custom domains (Team)

Team plans support fully custom domains like `status.yourapp.com`. Point a CNAME record to `status-proxy.crontinel.com` and add the domain in **Status Pages → Settings → Custom Domain**. SSL is provisioned automatically. DNS propagation usually takes under 10 minutes.
