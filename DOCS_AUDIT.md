# Crontinel Docs Launch Audit

Date: 2026-04-08
Auditor: Claude Code (claude-sonnet-4-6)
Prior art: /Users/ray/Work/crontinel/TRUST_AUDIT.md (do not duplicate findings already there)

---

## Scope

Every file under `docs/src/content/docs/` was read in full:

- `introduction.md`
- `quick-start.md`
- `installation.md`
- `troubleshooting.md`
- `self-hosting.md`
- `monitors/cron.md`
- `monitors/horizon.md`
- `monitors/queues.md`
- `alerts/channels.md`
- `alerts/cli.md`
- `reference/api.md`
- `reference/configuration.md`
- `mcp/overview.md`
- `mcp/claude-code.md`
- `mcp/tools.md`

---

## 1. Quick-Start / Installation Gaps

### Can a Laravel developer get their first monitor pinging in under 10 minutes?

Short answer: probably not on first try. Here is why.

**Gap QS-1 — No scheduler prerequisite step (P0, carried from TRUST_AUDIT)**

`quick-start.md` goes install → migrate → open dashboard, but never tells the developer to add `schedule:run` to their crontab. The cron section of the dashboard will be empty until at least one scheduler cycle fires. A developer following the quick-start exactly will hit step 4, see an empty cron table, and conclude the package is broken.

Fix: Add an explicit step 3.5 between "run migrations" and "open the dashboard":

```
### Make sure the Laravel scheduler is running

Add this to your server's crontab (`crontab -e`):

    * * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1

For local development with Valet or Herd, run:

    php artisan schedule:work
```

**Gap QS-2 — No "what you should see" after opening the dashboard (P0, new)**

After step 4 ("Visit /crontinel in your browser") there is no description or image showing what a fresh install looks like. The developer sees an empty dashboard and cannot tell if it is working correctly or broken. The troubleshooting doc handles post-install confusion but a first-time user will not know to go there.

Fix: Add one paragraph ("On a fresh install you will see the cron section empty until the scheduler fires once. Queue depth reads immediately. Horizon status appears if `laravel/horizon` is installed.") and a screenshot.

**Gap QS-3 — No note about `schedule:work` for local dev (P1, new)**

`schedule:run` in crontab only makes sense on a server. Developers running this locally with Valet, Herd, or plain `php artisan serve` will not have crontab set up. The quick-start should mention `php artisan schedule:work` (Laravel 8+) as the local alternative.

**Gap QS-4 — No "not using Horizon" note (P1, carried from TRUST_AUDIT)**

If `laravel/horizon` is not installed, the Horizon section of the dashboard will be empty. The quick-start has no note about this. Fix: one sentence after step 4 pointing to `'horizon' => ['enabled' => false]`.

**Gap QS-5 — Installation.md has no `config/crontinel.php` publish step shown (P1, new)**

`installation.md` tells you to run `crontinel:install` (which publishes the config) but never shows what the resulting `config/crontinel.php` looks like. A developer wanting to change any default has to go to the separate reference doc. At minimum a link to the configuration reference should appear here, or a trimmed example of the config.

**Gap QS-6 — No `.env` example file / copy-pasteable block in installation.md (P1, new)**

`installation.md` shows the `CRONTINEL_PATH` env var but not the full set of env vars. The configuration reference has the table, but a developer reading `installation.md` in isolation will not know what env vars to set. A 5-line `.env` block showing all the relevant vars (with blank defaults) would eliminate back-and-forth.

**Gap QS-7 — `self-hosting.md` references `github.com/crontinel/app.git` which may not be publicly accessible (P1, new)**

Step 1 of self-hosting runs `git clone https://github.com/crontinel/app.git`. The app repo is private (per TRUST_AUDIT context). If a user hits this before the repo is public or a Docker image exists, they are stuck on step 1 with no fallback.

Fix: Add a note saying "The self-hosting path requires access to the Crontinel app repo, which is available to Pro and Team plan subscribers. Contact support@crontinel.com for access." Or switch to a Docker-first self-hosting path.

---

## 2. Missing Code Snippets and Examples

**Gap CS-1 — No `routes/console.php` example showing `->monitor()` (P0, new)**

None of the docs show a code example of adding `->monitor()` to a scheduled command. This is a critical omission if Crontinel requires the developer to chain a method to their schedule definitions. If monitoring is fully automatic via event listeners (as `monitors/cron.md` implies), that should be stated explicitly and prominently in `quick-start.md` and `monitors/cron.md`. Right now these two things contradict: `monitors/cron.md` says "No code changes to your `schedule()` method are needed," but the OSS README may show otherwise. Clarify with a `routes/console.php` snippet either showing that no changes are needed (with existing commands visible) or showing any optional `->withoutOverlapping()` / custom tagging method if that exists.

**Gap CS-2 — `alerts/channels.md` says channels are added "in the Crontinel dashboard" but never shows the env-var / config-file fallback for OSS users (P0, new)**

The OSS package is self-contained with no SaaS dependency. Alert channel configuration for OSS users is done via `.env` and `config/crontinel.php`. But `alerts/channels.md` describes the Slack setup as: "add the channel in the Crontinel dashboard with config key `webhook_url`". This is SaaS-only language. An OSS-only user reading this page has no idea how to configure Slack alerts without the dashboard. The `reference/configuration.md` page has the correct config block, but `alerts/channels.md` does not link to it or show it.

Fix: Each channel section in `alerts/channels.md` needs a "Self-hosted / OSS" block showing the `.env` and `config/crontinel.php` snippet alongside the SaaS instructions.

**Gap CS-3 — PagerDuty and generic webhook channels have no env-var or config-file example at all (P1, new)**

`alerts/channels.md` documents Slack and email via env vars in the configuration reference, but PagerDuty and webhook have no env-var equivalents shown anywhere in the docs. If PagerDuty and webhooks are SaaS-only features, that needs to be stated. If they are supported in OSS, the config keys must be documented.

**Gap CS-4 — `self-hosting.md` missing queue worker queue names (P1, new)**

The Supervisor config in `self-hosting.md` runs `queue:work redis` with no `--queue` argument. For a production deploy you almost always need to specify which queues to work. The docs should show the actual queue names Crontinel uses internally (e.g., `crontinel`, `default`) so an operator can configure workers correctly.

**Gap CS-5 — `self-hosting.md` missing SSL/TLS details in Nginx config (P2, new)**

The Nginx block shows `listen 443 ssl` but no `ssl_certificate` or `ssl_certificate_key` directives. A developer copying this will get an nginx config error. Either show the cert lines or use a Certbot comment, or replace with the Caddy example (which handles TLS automatically) as the preferred path.

**Gap CS-6 — No example of a complete working `config/crontinel.php` file (P1, new)**

`reference/configuration.md` shows each key in isolation. There is no single block showing the entire default config file as published by `crontinel:install`. Developers who want to see the full shape of the config (to copy-paste into a PR or onboard a teammate) have to mentally assemble all the fragments. A single "full example" block at the top or bottom of the configuration reference page would dramatically help.

---

## 3. Missing Visuals

These are listed in impact order. All are new (not in TRUST_AUDIT).

**Gap V-1 — "Fresh install" dashboard screenshot (P0)**

What does `/crontinel` look like on day one, before any cron jobs have fired? Developers need to know the empty state is normal. A screenshot with callouts ("Cron section populates after the first scheduler run", "Queue depth reads immediately") would eliminate the most common first-run confusion.

**Gap V-2 — "Healthy dashboard" screenshot (P0)**

What does a fully populated, all-green dashboard look like? This is needed at the end of `quick-start.md` as the "you made it" moment, and in `introduction.md` to show what the product does before anyone installs anything.

**Gap V-3 — "Active alert" screenshot (P1)**

What does a firing alert look like in the dashboard? Users need to see the visual before they trust that alerts work. This belongs in `alerts/channels.md` and potentially in `monitors/cron.md` and `monitors/queues.md`.

**Gap V-4 — Architecture diagram (P1)**

`introduction.md` describes the three data sources (Horizon Redis keys, queue driver, scheduler events) but there is no diagram. A simple boxes-and-arrows diagram showing "Your Laravel App" → "crontinel/laravel package" → "local dashboard" and optionally → "app.crontinel.com" would make the architecture immediately clear, especially for the OSS-only vs SaaS comparison.

**Gap V-5 — "Late cron" vs "failed cron" visual distinction (P2)**

`monitors/cron.md` defines `late` and `failed` status but they look similar in prose. A screenshot or annotated table showing what each status looks like in the dashboard would help developers configure `late_alert_after_seconds` correctly.

**Gap V-6 — Horizon supervisor breakdown screenshot (P2)**

`monitors/horizon.md` explains per-supervisor tracking and calls it "the key difference from generic monitors." A screenshot showing two supervisors, one healthy and one down, would make this differentiator concrete and convincing.

---

## 4. Missing Docs Sections

**Gap D-1 — No FAQ page (P1, new)**

The most predictable questions that land in support are:
- "Do I need a Crontinel account to use this?" (no, OSS is standalone)
- "Will this slow down my app?" (no, events are async)
- "Does it work with Laravel Octane?" (unknown — should be answered)
- "What happens if app.crontinel.com goes down?" (local monitoring continues, SaaS sync resumes when it comes back)
- "Is the data sent to crontinel.com?" (only if you add an API key)
- "What does the free plan include?" (needs a clear answer before launch)

None of these are answered in a single place. Each answer exists implicitly somewhere in the docs but no FAQ surfaces them. For launch, a short FAQ of 6-8 entries would significantly reduce support volume.

**Gap D-2 — No billing / pricing explanation in docs (P1, new)**

The API reference mentions "Pro and Team plans" but there is no docs page explaining what each plan includes, what the free tier limits are, or how billing works. A developer deciding whether to stay OSS-only or sign up for SaaS needs this. Even a single paragraph with a link to the pricing page would be enough.

**Gap D-3 — No "upgrading the package" guide (P1, new)**

`self-hosting.md` has an upgrade section for the SaaS stack, but there is no guide for upgrading the `crontinel/laravel` package itself. Specifically: are migrations additive only? Will `crontinel:install` overwrite a customized `config/crontinel.php`? What changed between versions? This is a common source of friction on first upgrade.

Fix: Add a short section to `installation.md` (or a separate upgrade page) covering: `composer update crontinel/laravel`, checking if new migrations exist, and whether to re-run `crontinel:install --force`.

**Gap D-4 — No "Security" section or page (P1, new)**

The dashboard contains live infrastructure data (queue depths, job output, Horizon state). The docs have no guidance on:
- Hardening dashboard middleware in production (the troubleshooting doc shows how to add `can:view-monitoring` but frames it as optional)
- Whether the `crontinel_cron_runs` table captures sensitive output
- Data the package sends to the SaaS (if any, when no API key is set)
- Webhook HMAC signing (mentioned briefly in `mcp/tools.md` but never in `alerts/channels.md`)

A brief security page or section in `installation.md` would address the most common concerns from security-conscious teams.

**Gap D-5 — `self-hosting.md` has no Docker / Docker Compose option (P2, new)**

Self-hosting via git clone + manual server setup is the highest-friction option. A `docker-compose.yml` example (or even a link to one in the repo) would dramatically lower the bar for trying the self-hosted stack. Many evaluators will spin up a Docker compose to test before committing to a server install.

**Gap D-6 — No "Data retention and storage" explanation (P2, new)**

`monitors/cron.md` mentions `retain_days: 30` but there is no explanation anywhere of what data is stored, where, and for how long across the three monitors. Developers at larger companies often need this to assess compliance requirements (GDPR, SOC 2) before installing. A short "What data does Crontinel store?" section in `introduction.md` or a dedicated page would handle this.

**Gap D-7 — `alerts/channels.md` has no "alert routing" or "silence windows" section (P2, new)**

There is no mention of whether alerts can be silenced (maintenance windows), routed to different channels by monitor type, or throttled beyond the 5-minute dedup window. If none of these features exist yet, the docs should say so explicitly so developers know not to wait for them.

---

## 5. Recommended Fix Order

Ranked by user impact for a developer trying to get their first monitor working.

| Rank | Gap ID | Title | Impact | Effort |
|------|--------|-------|--------|--------|
| 1 | QS-1 | Add scheduler prerequisite step to quick-start | P0 — first-run blocker | 20 min |
| 2 | QS-2 | Add "what you should see" paragraph after dashboard step | P0 — removes confusion about empty state | 15 min |
| 3 | CS-2 | Add OSS alert channel config examples to alerts/channels.md | P0 — OSS users can't configure alerts from that page | 30 min |
| 4 | QS-4 | Add "not using Horizon?" note to quick-start | P1 | 10 min |
| 5 | QS-3 | Add `schedule:work` local dev note | P1 | 10 min |
| 6 | CS-1 | Clarify routes/console.php and whether `->monitor()` is needed | P0 | 30 min |
| 7 | CS-6 | Add full config/crontinel.php example block to configuration reference | P1 | 20 min |
| 8 | QS-6 | Add env var block to installation.md | P1 | 15 min |
| 9 | V-1 | Add fresh-install dashboard screenshot to quick-start | P0 | 30 min (screenshot + embed) |
| 10 | V-2 | Add healthy dashboard screenshot to introduction.md | P0 | 30 min |
| 11 | D-1 | Write FAQ page (6-8 entries) | P1 | 45 min |
| 12 | D-2 | Add billing/plan explanation (even just a paragraph + link) | P1 | 15 min |
| 13 | D-3 | Add package upgrade guide to installation.md | P1 | 20 min |
| 14 | D-4 | Add Security section to installation.md | P1 | 30 min |
| 15 | CS-3 | Document PagerDuty + webhook config keys for OSS | P1 | 20 min |
| 16 | CS-4 | Add queue names to self-hosting Supervisor config | P1 | 10 min |
| 17 | V-3 | Add active-alert screenshot to alerts/channels.md | P1 | 30 min |
| 18 | V-4 | Add architecture diagram to introduction.md | P1 | 1 hr |
| 19 | QS-7 | Clarify self-hosting repo access requirement | P1 | 15 min |
| 20 | D-5 | Add Docker Compose self-hosting option | P2 | 2 hr |
| 21 | CS-5 | Fix Nginx SSL block (add cert directives or note) | P2 | 10 min |
| 22 | D-6 | Add data retention / storage explanation | P2 | 20 min |
| 23 | D-7 | Document silence windows / alert routing (or explicitly note absence) | P2 | 15 min |
| 24 | V-5 | Add late vs failed cron screenshot | P2 | 30 min |
| 25 | V-6 | Add Horizon supervisor breakdown screenshot | P2 | 30 min |

---

## What the Docs Do Well (Do Not Break)

- `troubleshooting.md` is excellent: specific, ordered, with exact commands. It is the strongest page in the docs.
- `monitors/horizon.md` clearly explains the per-supervisor vs top-level status distinction — this is the key product differentiator and it is communicated well.
- `reference/configuration.md` and `reference/api.md` are complete and well-structured.
- `mcp/tools.md` is thorough with parameter tables and a worked example.
- `alerts/channels.md` handles deduplication and auto-resolution clearly.

---

## Total Time Estimate (P0 + P1 only)

All P0 and P1 gaps can be closed in approximately 6-8 hours of focused writing and screenshotting. P2 items add another 4-5 hours. The highest-value hour is the first one: fix QS-1, QS-2, QS-3, QS-4 together (all in `quick-start.md`) and the scheduler-blocker is gone.
