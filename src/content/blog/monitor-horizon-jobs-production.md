---
title: "How to Monitor Horizon Jobs in Production"
description: "Laravel Horizon gives you a beautiful dashboard and powerful job management tools — but by default it will not alert you when something goes wrong. Here is how to build a monitoring layer that catches Horizon problems before they become production incidents."
date: 2026-04-17
tags: ["laravel", "horizon", "queues", "monitoring", "production"]
---

Laravel Horizon is one of those tools that makes you wonder how you ever shipped a Laravel application without it. It gives you a real-time view of your queues, a convenient way to balance workers across servers, and a visual history of every job that has passed through your system. For teams running heavy queue workloads, it is close to essential.

But Horizon has a problem that trips up a lot of teams: it is primarily a management interface, not a monitoring system. You can open the dashboard and see that something is wrong, but Horizon will not send you a message at 2 AM when 3,000 jobs have been sitting in a queue for twenty minutes. You have to be looking.

This is the gap this post fills. By connecting Horizon to a proper monitoring layer, you get proactive alerts on the metrics that matter — without losing Horizon's excellent management UX.

## What Horizon Does Well Out of the Box

Before adding monitoring, it is worth being clear about what Horizon already gives you for free.

**The dashboard** shows live job throughput, failed jobs, recent job batches, and worker status across all your servers. If you have a second monitor and the discipline to glance at it during the day, the dashboard is genuinely useful for spotting anomalies.

**Job tagging** lets you group jobs by purpose and see throughput per tag. Tags also enable targeted worker allocation — you can dedicate a subset of workers to specific job types without affecting others.

**Batches** give you visibility into grouped job execution. When you dispatch 10,000 email jobs as a batch, Horizon tracks the progress, tells you how many have completed, and reports back when the whole batch is done.

**Failed job tracking** stores exception details for every failed job, with full stack traces accessible from the UI.

What Horizon does not do is notify you when any of this goes wrong. The dashboard assumes someone is watching it.

## The Production Monitoring Gap

Here is the failure scenario that plays out repeatedly in Laravel applications running Horizon.

A deployment ships a breaking change to an email template. The email queue starts failing — not because the mail server is down, but because the template expects a variable that is no longer present. Horizon marks the jobs as failed and stores the exceptions. The queue depth climbs. Ten minutes later, 1,200 users have signed up and none of them have received their welcome emails. No page crashed. No HTTP response errored. Horizon logged everything correctly; it just did not tell anyone.

This is the pattern. Horizon is excellent at recording what happened. It is passive about telling you it happened. Monitoring fills that gap.

## Key Metrics to Watch on Horizon

Effective Horizon monitoring covers the same dimensions as general queue monitoring, with a few Horizon-specific additions.

**Queue depth** — How many jobs are waiting in each queue? Horizon exposes this through its internal metrics system. A sudden spike in waiting jobs almost always means workers are falling behind or have stopped.

**Recent job failures** — Failed jobs are the most actionable metric. A handful of failures from transient errors is normal. A sudden cluster of identical failures means something systemic broke.

**Batches in progress** — Long-running batches that are not completing are a sign of jobs stuck in a retry loop or workers that are not processing the batch queue.

**Terminated workers** — Horizon manages worker processes. If a supervisor process crashes or is killed, Horizon restarts it — but you should know this happened. Repeated worker restarts can indicate memory issues or an unstable deployment.

**Job wait time** — How long does a job sit in the queue before a worker picks it up? If wait times are growing even when worker count is stable, something is consuming worker capacity.

## Setting Up Crontinel to Monitor Horizon

Crontinel connects directly to Horizon's Redis metrics, so you do not need to instrument your jobs or modify your application code.

```bash
composer require crontinel/laravel
php artisan crontinel:install --horizon
```

The `--horizon` flag configures Crontinel to read from Horizon's metric tags rather than the generic Laravel queue interface. After installation, Crontinel automatically discovers your Horizon-supervised queues and begins collecting metrics.

```php
// config/crontinel.php
return [
    'enabled' => true,
    'horizon' => [
        'enabled' => true,
        'redis_connection' => 'default',
        'metrics_ttl' => 3600,       // How long to retain metrics in Redis
    ],
    'queues' => [
        'emails' => [
            'alert_threshold' => 50,
            'slow_job_threshold' => 15,
            'failure_alert_threshold' => 3,
        ],
        'default' => [
            'alert_threshold' => 200,
            'slow_job_threshold' => 30,
            'failure_alert_threshold' => 10,
        ],
        'notifications' => [
            'alert_threshold' => 100,
            'failure_alert_threshold' => 5,
        ],
    ],
];
```

## Alerting on Horizon Failures

Crontinel uses Horizon's event system to detect failures in real time, without polling.

```php
// config/crontinel.php
'alerts' => [
    'slack' => [
        'webhook_url' => env('CRONTINEL_SLACK_WEBHOOK'),
        'channel' => '#horizon-alerts',
        'throttle_minutes' => 5,
    ],
    'pagerduty' => [
        'integration_key' => env('PAGERDUTY_KEY'),
        'severity' => 'error',
    ],
],
```

The alert payload includes the job name, exception type, the queue it was dispatched to, and a link directly to the failed job in Horizon's dashboard. This means whoever gets paged can go straight to the context, not spend ten minutes reproducing the error.

## Investigating Failures Through Horizon

When an alert fires, Horizon's dashboard is the fastest way to understand what went wrong.

The **Failed Jobs** section shows every failed job with the exception message, stack trace, and the payload that was passed to the job. For jobs that failed in a batch, Horizon shows which jobs in the batch failed and which succeeded.

If failures are clustering around a specific job class, check the recent deployment. A template error, a missing configuration value, or a renamed method will usually surface immediately when you look at the failure payload.

For transient failures — network timeouts, rate limiting from external services — Horizon's built-in retry command handles re-queuing cleanly:

```bash
php artisan horizon:retry
```

This retries all failed jobs. You can also target a specific job by ID:

```bash
php artisan horizon:retry connection=redis,queue=emails --failed
```

## Balancing Workers Across Servers

One of Horizon's most valuable features is its tag-based worker balancing. You can allocate a fixed number of workers to specific job tags, which is useful when certain job types need guaranteed throughput.

```php
// config/horizon.php
'environments' => [
    'production' => [
        'supervisor-1' => [
            'connection' => 'redis',
            'queue' => ['emails', 'default'],
            'balance' => 'auto',
            'processes' => 10,
            'tries' => 3,
            'timeout' => 60,
        ],
        'supervisor-2' => [
            'connection' => 'redis',
            'queue' => ['notifications'],
            'balance' => 'simple',
            'processes' => 5,
            'tries' => 3,
            'timeout' => 90,
        ],
    ],
],
```

The `balance => 'auto'` setting lets Horizon scale worker processes up and down based on queue depth. When combined with Crontinel monitoring, you get both automatic scaling and proactive alerting — Horizon manages capacity while Crontinel makes sure someone knows when the queue is backing up despite the scaling.

## Knowing When Horizon Itself Is the Problem

Horizon is a PHP process running on your server. Like any process, it can consume too much memory, leak connections, or get OOM-killed by the OS. When Horizon goes down, your queues stop being processed and no failures are recorded — the jobs just sit in Redis.

Crontinel monitors Horizon process health separately from job health:

```php
// config/crontinel.php
'horizon' => [
    'monitor_process' => true,
    'process_down_alert_minutes' => 2,  // Alert within 2 minutes if Horizon stops
],
```

A process-down alert is the most urgent notification you can receive. Unlike a job failure, which means some work did not complete, a Horizon-down event means all queue processing has halted. Every minute counts.

## Preventing Horizon Surprises

A few practices keep Horizon healthy and reduce firefighting in production.

**Deploy during low-traffic windows** — Horizon restarts its supervisors on deployment. A rolling restart with `horizon:terminate` gracefully finishes current jobs before restarting, but it is still worth timing deployments when queue depth is low.

**Set memory limits on supervisors** — Horizon's configuration accepts a `memory` limit that restarts a supervisor process when it exceeds the threshold. Without this, a memory leak in a job will slowly consume available RAM until the process dies unexpectedly.

**Monitor Redis, not just Horizon** — Horizon uses Redis to store jobs, tags, and metrics. If Redis becomes unavailable or reaches connection limits, Horizon cannot dispatch or process jobs. Redis monitoring is a prerequisite for Horizon reliability.

**Use Crontinel's trend view** — Queue depth trends tell you whether your queues are growing over time even when individual measurements look acceptable. A queue that grows 20% per week is heading toward an incident even if today's numbers look fine.

## Get Started

Horizon gives you the tools to manage complex queue workloads. Monitoring makes sure you know when those tools need your attention.

The [quick start guide](/docs/quick-start) covers installing Crontinel alongside Horizon and configuring your first alerts in under ten minutes. If you are already running Horizon, the integration takes about five minutes — it reads from the same Redis connection Horizon uses, with no code changes required.

Your queues have been running in the background this whole time. It is time to start listening.

[Start monitoring Horizon in production →](/docs/quick-start)
