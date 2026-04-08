# Crontinel Docs — Launch Gaps Summary

Date: 2026-04-08
Full audit at: `docs/src/content/docs/DOCS_AUDIT.md`

---

## Can a Laravel developer get their first monitor pinging in under 10 minutes?

Not reliably, on first try. Three blockers stand between "composer require" and "sees data in the dashboard":

1. The scheduler (`schedule:run`) is never mentioned in the quick-start. Empty cron section on first load looks like a bug.
2. No "what you should see" step after opening `/crontinel`. Developer cannot tell empty state from broken state.
3. `alerts/channels.md` describes alert setup using SaaS dashboard language only. OSS users have no idea how to configure Slack alerts from that page.

Fix those three and the 10-minute path is clear.

---

## P0 — Must fix before launch

| ID | File | Gap |
|----|------|-----|
| QS-1 | `quick-start.md` | Add `schedule:run` crontab step and `schedule:work` for local dev |
| QS-2 | `quick-start.md` | Add "what you should see" paragraph after dashboard step (empty state is normal) |
| CS-1 | `quick-start.md`, `monitors/cron.md` | Clarify whether `->monitor()` in `routes/console.php` is required or not; show a snippet either way |
| CS-2 | `alerts/channels.md` | Add OSS `.env` / `config/crontinel.php` config block for every channel (Slack, email) alongside SaaS instructions |
| V-1 | `quick-start.md` | Fresh-install dashboard screenshot with callout "Cron section populates after first scheduler run" |
| V-2 | `introduction.md` | Healthy all-green dashboard screenshot |

---

## P1 — Fix in launch week

| ID | File | Gap |
|----|------|-----|
| QS-3 | `quick-start.md` | `php artisan schedule:work` note for local dev |
| QS-4 | `quick-start.md` | "Not using Horizon? set `enabled => false`" note |
| QS-5 | `installation.md` | Link to configuration reference after install step |
| QS-6 | `installation.md` | Full env-var block (all vars, blank defaults) |
| QS-7 | `self-hosting.md` | Note that app repo access requires Pro/Team plan (it is private) |
| CS-3 | `alerts/channels.md` | Document PagerDuty and webhook config keys for OSS (or state SaaS-only) |
| CS-4 | `self-hosting.md` | Add Crontinel's internal queue names to Supervisor config |
| CS-6 | `reference/configuration.md` | Add a single "full config/crontinel.php" example block at top of page |
| D-1 | new `faq.md` | FAQ: free vs paid, data sent to SaaS, Octane support, what happens if SaaS goes down |
| D-2 | `introduction.md` or new | One paragraph on plans/billing with link to pricing page |
| D-3 | `installation.md` | Package upgrade guide (composer update, new migrations, re-publish config?) |
| D-4 | `installation.md` | Security section: hardening middleware, sensitive output in cron table, webhook HMAC |
| V-3 | `alerts/channels.md` | Active alert screenshot |
| V-4 | `introduction.md` | Architecture diagram (Laravel app → package → local dashboard → optional SaaS) |

---

## P2 — Pre-launch polish

| ID | File | Gap |
|----|------|-----|
| D-5 | `self-hosting.md` | Docker Compose self-hosting path |
| D-6 | `introduction.md` | Data retention and storage explanation |
| D-7 | `alerts/channels.md` | Silence windows / alert routing (or state these don't exist yet) |
| CS-5 | `self-hosting.md` | Fix Nginx config (missing ssl_certificate directives) |
| V-5 | `monitors/cron.md` | Late vs failed cron screenshot |
| V-6 | `monitors/horizon.md` | Per-supervisor breakdown screenshot |

---

## What the docs do well — do not change

- `troubleshooting.md` is the strongest page: specific, ordered, exact commands. Keep it as-is.
- `monitors/horizon.md` explains per-supervisor vs top-level status clearly — this is the key product differentiator.
- `reference/configuration.md` and `reference/api.md` are complete.
- `mcp/tools.md` is thorough with parameter tables and a worked example.

---

## Time estimate

| Tier | Gaps | Estimated effort |
|------|------|-----------------|
| P0 (6 items) | Quick-start text fixes + 2 screenshots | ~2.5 hours |
| P1 (14 items) | New sections, FAQ page, missing snippets | ~4 hours |
| P2 (6 items) | Docker, diagrams, polish | ~4 hours |

The first 90 minutes (QS-1 through QS-4 + CS-2 in `quick-start.md` and `alerts/channels.md`) eliminates the three first-run blockers. That is the highest-leverage work before launch.
