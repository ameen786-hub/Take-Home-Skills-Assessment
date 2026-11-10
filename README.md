# Automation & QA Developer — Take-Home

## Contents
| File | What it is |
|---|---|
| `Task1_QA_Report_YourName.docx` | Bug table (8 candidate issues) + root-cause analysis |
| `Task2_Workflow_YourName.json` | n8n "Morning Brief" workflow |
| `Bonus_UptimeMonitor_YourName.json` | n8n uptime monitor (5 min) |
| `screenshots/` | Canvas + execution screenshots (add after running) |

## Task 2 — Morning Brief
**APIs:** GitHub REST API (public, no key needed) for both calls.
1. `GET /search/repositories?q=topic:automation&sort=stars` — one call returns 20 popular repos with star counts.
2. `GET /repos/{full_name}/releases/latest` — per-repo enrichment (latest release tag). Many repos have no releases, so this naturally returns 404 and exercises the error path.

**Why GitHub:** no signup, stable schema, and star counts give a natural numeric threshold.

**Transformation (Code node):** sort by stars, keep top 5, trim description to 120 chars.

**Branch:** IF `stars > 1000` → 🔥 Hot, else 👀 Watch. Both branches merge into one Discord message.

**Output:** Discord webhook (via n8n Credentials, type *Discord Webhook API*; no URL in the JSON).

**Error handling**
- Search call: 3 retries, then the *error output* sends a Discord alert instead of failing silently.
- Release call: *Continue (regular output)*. A 404 shows "none / unavailable"; any other failure adds ⚠️ to that line, so one bad item never blocks the digest.
- A separate **Error Trigger** branch alerts on any unexpected node failure.

## Bonus — Uptime monitor
Every 5 min → HTTP GET (10 s timeout, 3 tries, `neverError` so non-200 returns a status) → Code records response time and counters in workflow static data → IF alert needed → Discord. Alerts fire on state change (DOWN, then RECOVERED) to avoid spam. A 09:00 daily schedule posts checks, uptime %, avg/max response time.
Note: static data persists only when the workflow is **active**, not in manual test runs.

## Setup
1. Import both JSON files into n8n.
2. Create credential *Discord Webhook API* and select it on each Discord node (placeholder id `REPLACE_ME`).
3. Run Task 2 manually; for the monitor, set `target_url` in the **Config** node, then activate.

## Assumptions / not verified
- The workflows were authored and structurally validated (valid JSON, all links resolve) but **not executed** in an n8n instance. The Discord node's parameter names (typeVersion 2, webhook auth) may need a quick re-select on import.
- The QA findings are unconfirmed candidates until reproduced (see banner in the report).
