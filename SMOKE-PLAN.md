# Smoke plan — one KNS domain on TN10

1. Ensure funding: backup address has TKAS (api-tn10 balance).
2. `POST /api/v1/domains/check` for `smoke-kns-tn10-001.kas` (or new unique label) with payer address.
3. If available: create via KasWare UI on https://tn10.knsdomains.org/ **or** scripted commit/reveal paying fee to KNS TN10 sink.
4. Wait for indexer; `GET /api/v1/smoke-....kas/owner` must return our address.
5. Log txids + any KasWare/UI errors into FINDINGS-DRAFT.md.
6. Only then scale (batches of check 100, slow reveals).
