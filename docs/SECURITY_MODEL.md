# Security model

## Authorization

Scan only projects the operator owns or is authorized to assess. Uploaded archives are treated as untrusted files for static analysis, not as code to execute.

## Upload and extraction

- Only `.zip`
- Size cap: `MAX_UPLOAD_SIZE_MB` / `MAX_UPLOAD_SIZE`
- Zip Slip blocked via resolved-path `relative_to` checks
- Zip bombs blocked by uncompressed size
- Symlink members rejected
- Extracted under `UPLOAD_DIRECTORY / {scan_id}`

## Scanner execution

- `subprocess.run([...], shell=False)` argument arrays
- Timeouts on optional CLIs
- Missing CLIs return empty findings plus a setup instruction
- Uploaded projects are never installed, built, or executed

## Secrets

- API keys stay in backend env
- Frontend talks only to `/api`
- Logs should not print file contents that look like secrets (evidence snippets are truncated in the native secret rules)

## CORS

Configured from `FRONTEND_URL` plus local Vite origins. Tighten `allow_origins` for any shared deployment (do not ship with `*`).

## Remediation

No automatic patch deploy, credential rotation, or remote change. Status moves only after `approve` and `verify` API calls.

## Uncertainty

AI threat text is labelled as analysis. Scanner evidence, public KEV records, Hindsight memories, and hypotheses are shown separately. PHP2Shell findings are configuration/version evidence of exposure, not a claim that a specific host was exploited.
