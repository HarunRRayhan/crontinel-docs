---
title: Quick Start
description: Get Crontinel running in under 5 minutes
---

## 1. Install the package

```bash
composer require harunrrayhan/crontinel
```

## 2. Run the installer

```bash
php artisan crontinel:install
```

This publishes `config/crontinel.php` and creates the `crontinel_runs` migration.

## 3. Run migrations

```bash
php artisan migrate
```

## 4. Open the dashboard

Visit `/crontinel` in your browser. You'll see the monitoring dashboard.

## 5. (Optional) Configure alerts

In `.env`:

```env
CRONTINEL_ALERT_CHANNEL=slack
CRONTINEL_SLACK_WEBHOOK=https://hooks.slack.com/services/...
```

## 6. (Optional) Connect to the SaaS

Create a free account at [app.crontinel.com](https://app.crontinel.com), create an app, and add your API key:

```env
CRONTINEL_API_KEY=your-api-key-here
```
