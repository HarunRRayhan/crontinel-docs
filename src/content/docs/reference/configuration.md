---
title: Configuration Reference
description: All config/crontinel.php options
---

After running `php artisan crontinel:install`, edit `config/crontinel.php`:

## `path`

Dashboard URL path. Default: `crontinel`

```php
'path' => env('CRONTINEL_PATH', 'crontinel'),
```

## `middleware`

Middleware applied to the dashboard routes. Default: `['web', 'auth']`

```php
'middleware' => ['web', 'auth'],
```

## `horizon`

```php
'horizon' => [
    'enabled' => true,
    'supervisor_alert_after_seconds' => 60,    // how long before a dead supervisor fires an alert
    'failed_jobs_per_minute_threshold' => 5,   // alert when failed rate exceeds this
    'connection' => 'horizon',                  // Redis connection Horizon uses
],
```

## `queues`

```php
'queues' => [
    'enabled' => true,
    'watch' => [],                     // specific queue names; empty = auto-discover
    'depth_alert_threshold' => 1000,   // alert when a queue exceeds this depth
    'wait_time_alert_seconds' => 300,  // alert when oldest job is older than this
],
```

## `cron`

```php
'cron' => [
    'enabled' => true,
    'late_alert_after_seconds' => 120, // mark run as late after this many seconds
    'retain_days' => 30,               // how long to keep cron run history
],
```

## `alerts`

```php
'alerts' => [
    'channel' => env('CRONTINEL_ALERT_CHANNEL'), // 'slack' or 'mail'
    'mail'  => ['to' => env('CRONTINEL_ALERT_EMAIL')],
    'slack' => ['webhook_url' => env('CRONTINEL_SLACK_WEBHOOK')],
],
```

## Environment variables

| Variable | Default | Description |
|---|---|---|
| `CRONTINEL_PATH` | `crontinel` | Dashboard URL path |
| `CRONTINEL_ALERT_CHANNEL` | — | Alert channel: `slack` or `mail` |
| `CRONTINEL_SLACK_WEBHOOK` | — | Slack incoming webhook URL |
| `CRONTINEL_ALERT_EMAIL` | — | Email alert recipient |
| `CRONTINEL_API_KEY` | — | SaaS API key (optional) |
| `CRONTINEL_API_URL` | `https://app.crontinel.com` | SaaS URL |
