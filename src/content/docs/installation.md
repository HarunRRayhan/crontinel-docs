---
title: Installation
description: Full installation and configuration guide
---

## Requirements

- PHP 8.2, 8.3, or 8.4
- Laravel 11, 12, or 13

## Install

```bash
composer require crontinel/laravel
php artisan crontinel:install
php artisan migrate
```

After running the installer, `config/crontinel.php` is published to your project. Review it to set the dashboard path, middleware, and alert thresholds. See the [configuration reference](/reference/configuration) for all available options.

## Dashboard path

By default the dashboard is at `/crontinel`. To change it:

```env
CRONTINEL_PATH=monitoring
```

## Middleware

The dashboard uses `['web', 'auth']` middleware by default. To customize:

```php
// config/crontinel.php
'middleware' => ['web', 'auth', 'can:view-monitoring'],
```

## Requirements check

```bash
php artisan crontinel:check
```

Exits 0 if healthy, 1 if any alert is active. Use in CI pipelines:

```bash
php artisan crontinel:check --format=json
```

## Environment variables

All Crontinel `.env` variables:

```env
# Dashboard
CRONTINEL_PATH=crontinel

# Alerts — set CRONTINEL_ALERT_CHANNEL to one of: slack, mail, pagerduty, webhook
CRONTINEL_ALERT_CHANNEL=
CRONTINEL_SLACK_WEBHOOK=
CRONTINEL_ALERT_EMAIL=
CRONTINEL_PAGERDUTY_ROUTING_KEY=
CRONTINEL_WEBHOOK_URL=

# SaaS (optional)
CRONTINEL_API_KEY=
CRONTINEL_API_URL=https://app.crontinel.com
```
