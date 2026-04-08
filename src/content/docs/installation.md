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

After running the installer, `config/crontinel.php` is published to your project. Review it to set the dashboard path, middleware, and alert thresholds.

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

Commonly used `.env` variables:

```env
CRONTINEL_PATH=crontinel
CRONTINEL_ALERT_CHANNEL=slack
CRONTINEL_SLACK_WEBHOOK=https://hooks.slack.com/services/YOUR/WEBHOOK/URL
# Or for email alerts:
CRONTINEL_ALERT_EMAIL=you@example.com
```
