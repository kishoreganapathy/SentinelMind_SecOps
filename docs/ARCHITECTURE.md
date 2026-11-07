# SentinelMind Architecture

This document describes the implementation in this workspace.

## Layout

```
backend/app/
  main.py                 FastAPI app, DB init, knowledge + demo project seed
  config.py               pydantic-settings from backend/.env
  cli.py                  ingest-knowledge, seed-demo-memory
  api/                    REST routers under /api
  agents/                 LangGraph orchestrator and six agents
  services/scanners/      native + Semgrep/Gitleaks/OSV/Trivy + zip safety
  services/memory/        official Hindsight client + labelled mock adapter
  services/knowledge/     public CVE/CISA/MITRE store
  services/correlation/   attack-path engine
  services/prioritization/deterministic risk formula
  services/remediation/   advisor + verification retain
  fixtures/synthetic_php_project/  labelled synthetic 2026 PHP benchmark
frontend/src/
  pages/                  seven dashboard surfaces
  services/api.ts         /api client
```

## Request flow

1. `POST /api/scans` extracts a ZIP safely or resolves a local path (including the synthetic fixture).
2. `ScannerAgent` runs selected scanners with subprocess argument arrays (never shell strings).
3. `FindingNormalizer` fingerprints file + category + CWE/CVE/line bucket and merges evidence.
4. `POST /api/investigations` or `/api/investigations/compare` runs LangGraph:
   threat analysis → historical intelligence → correlation → risk → remediation → synthesis.
5. Historical intelligence queries SQLite/Postgres public knowledge in both modes.
   Hindsight `recall` runs only in `with_memory`.
6. `POST /api/remediations/{id}/approve` requires a human identity.
7. `POST /api/remediations/{id}/verify` updates finding status and `retain`s the outcome.

## Data stores

- **SQLAlchemy** (`DATABASE_URL`): projects, scans, findings, investigations, remediations, public knowledge.
- **Hindsight bank** (`HINDSIGHT_BANK_ID`): investigation memories and lessons. SQL does not clone the full memory graph.

## Risk formula (deterministic)

Documented in `RiskPrioritizationEngine`:

```
finding_score =
  0.40 * severity * confidence
+ 0.25 * exploitability (CISA KEV / PHP2Shell / injection)
+ 0.20 * chained-path multiplier
+ 0.15 * Hindsight corroboration
```

Project score = `0.50 * max + 0.30 * avg(top 5) + 0.20 * chain penalty`.

## PHP 2026 detection

Native engine flags:

- Unpatched PHP images / Composer platform constraints below 8.2.35 / 8.3.36 / 8.4.27 / 8.5.12
- `cgi.fix_pathinfo=1` → `CVE-2026-109618`
- `session.upload_progress.enabled=On` → `CVE-2026-109619`
- php-cgi Action mappings and CVE-2024-4577 argument-injection patterns
- Application SQLi, unserialize, secrets, root DB user

Sources cited in findings: CISA alert 2026-10-21 and KEV add date 2026-10-26.
