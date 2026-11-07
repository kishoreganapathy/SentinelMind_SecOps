# Demo script (about 60 seconds)

Goal: show that Hindsight changes the investigation of the **same** PHP 2026 findings.

## Setup

1. `cd backend && python -m uvicorn app.main:app --host 127.0.0.1 --port 8000`
2. `cd frontend && npm run dev`
3. Open http://127.0.0.1:5173

Optional: start Hindsight on port 8888. If you skip it, say clearly that the Memory Center is on the labelled local adapter.

## Script

1. **Dashboard (10s)** — Total scans/findings, Hindsight status chip (Official vs Local Fallback).
2. **New Investigation (15s)** — Click **1-Click Scan Benchmark**. Explain the fixture is synthetic and contains:
   - PHP2Shell posture: PHP 8.2.20 image, `cgi.fix_pathinfo=1`, `session.upload_progress.enabled=On` (`CVE-2026-109618`, `CVE-2026-109619`)
   - php-cgi mapping / CVE-2024-4577
   - Hardcoded MySQL root password
   - SQLi and `unserialize()`
3. **Findings (10s)** — Filter CRITICAL. Point at PHP2Shell CVEs and the DB URI. Evidence is scanner output, not invented chat text.
4. **Before vs After Memory (20s)** — Run comparison.
   - Left: public CISA/NVD + current evidence. Plausible path, generic CGI advice.
   - Right: recalls synthetic incident `SEC-2026-110`. Confirmed chain, warns that WAF-only / upload_progress-only failed, prescribes patched PHP 8.2.35+ and credential rotation.
5. **Remediation (5s)** — Approve, then verify success. Memory Center shows a new retain. Say the next scan can recall that outcome.

## What not to claim

- Do not call `SEC-2026-110` a real customer breach. It is labelled synthetic.
- PHP2Shell CVEs themselves are public (CISA, Oct 2026); the org story around them is demo data.
- Do not run exploits against live hosts.
