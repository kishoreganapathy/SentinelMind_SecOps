# Hindsight Integration

Verified against the official Python client documentation:

- Install: `pip install hindsight-client`
- Docs: https://hindsight.vectorize.io/sdks/python
- Methods: `retain`, `recall`, `reflect` on `Hindsight(base_url=...)`

## What this project calls

```python
from hindsight_client import Hindsight
client = Hindsight(base_url=settings.HINDSIGHT_BASE_URL)
# optional api_key if the deployment requires it

client.retain(bank_id=..., content=..., context=..., metadata=..., tags=...)
client.recall(bank_id=..., query=..., max_tokens=4096)
client.reflect(bank_id=..., query=...)
```

Official `recall` does **not** take a `tags=` filter in the documented SDK. SentinelMind folds tags into the query string when talking to the live client. The local mock adapter still filters tags for tests.

Official `reflect` takes `query` (and budget). Situational context is appended to the query text rather than inventing unsupported kwargs.

`list_memories` / `get_version` are used only when present; failures fall back to the mock adapter or a port probe.

## Two knowledge layers

| Layer | Store | Provenance label |
| --- | --- | --- |
| Public cyber knowledge | SQL `historical_knowledge` | `PUBLIC_NVD_KEV` |
| Org experience | Hindsight bank | `Official Hindsight Memory Bank` or `Local Development Mock Memory (Hindsight Offline)` |

## Fallback

If the SDK is missing or `HINDSIGHT_BASE_URL` is unreachable:

- `service_type` = `LOCAL_MOCK_FALLBACK`
- JSON file: `./data/mock_hindsight_bank.json`
- UI: “Local Fallback Adapter Active”
- Scans and findings still work
- Comparison mode still recalls mock-seeded **labelled synthetic** incident `SEC-2026-110` so the demo is runnable without a daemon

Never present mock results as official Hindsight.

## Environment

```
HINDSIGHT_BASE_URL=http://localhost:8888
HINDSIGHT_API_KEY=
HINDSIGHT_BANK_ID=sentinelmind-secops
HINDSIGHT_TIMEOUT_SECONDS=3.0
```

Run a Hindsight daemon locally or point `HINDSIGHT_BASE_URL` at your hosted instance. Then restart the API and confirm `/api/health` shows `OFFICIAL_HINDSIGHT`.
