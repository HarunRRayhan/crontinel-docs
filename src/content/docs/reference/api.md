---
title: REST API Reference
description: Crontinel REST API for Pro and Team plan users
---

The REST API is available on Pro and Team plans. All endpoints require an API key.

## Authentication

Pass your API key as a Bearer token:

```
Authorization: Bearer your-api-key
```

Or as a header:

```
X-Api-Key: your-api-key
```

## Base URL

```
https://app.crontinel.com/api/v1
```

## Endpoints

### GET /apps

List all apps for the authenticated team.

**Response:**
```json
{
  "data": [
    {
      "name": "My App",
      "slug": "my-app",
      "status": "healthy",
      "last_ping_at": "2026-04-06T12:00:00Z"
    }
  ]
}
```

### GET /apps/{slug}/status

Full health snapshot for an app.

**Response:**
```json
{
  "app": {"name": "My App", "slug": "my-app"},
  "status": "healthy",
  "horizon": {"status": "running", "paused": false, "failed_per_minute": 0.2},
  "queues": [{"name": "default", "depth": 5, "failed": 0}],
  "active_alerts": [],
  "last_updated": "2026-04-06T12:00:00Z"
}
```

### GET /apps/{slug}/cron-runs

Last 50 cron runs, newest first.

**Response:**
```json
{
  "data": [
    {
      "command": "php artisan inspire",
      "status": "completed",
      "exit_code": 0,
      "duration_ms": 145,
      "started_at": "2026-04-07T08:00:02Z"
    }
  ]
}
```

| Field | Type | Notes |
|---|---|---|
| `command` | string | The artisan or shell command |
| `status` | string | `completed`, `failed`, `running`, or `late` |
| `exit_code` | int \| null | Process exit code (null if still running) |
| `duration_ms` | int \| null | Execution time in milliseconds |
| `started_at` | string | ISO 8601 timestamp |

### GET /apps/{slug}/alerts

Active (unresolved) alerts for the app.

**Response:**
```json
{
  "data": [
    {
      "alert_key": "queue:default:depth",
      "fired_at": "2026-04-07T07:55:00Z",
      "fire_count": 3
    }
  ]
}
```

| Field | Type | Notes |
|---|---|---|
| `alert_key` | string | Unique key like `horizon:paused`, `queue:default:depth`, `cron:send-invoices:failed` |
| `fired_at` | string | When the alert first fired |
| `fire_count` | int | How many times the condition was re-evaluated as firing |

## OpenAPI spec

Download the full spec at:

```
https://app.crontinel.com/openapi.json
```
