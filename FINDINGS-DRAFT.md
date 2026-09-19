# kns / KasWare TN10 test findings (draft)

Date: 2026-09-19 (Europe/Brussels / CEST)

Scope: **testnet-10 only**. Do not touch mainnet. Do not stop miner farm on `:16211` or `/tmp/kaspa-data-tn10`.

## Endpoints that work (probed live)

Base: `https://api.knsdomains.org/tn10`

| Method | Path | Notes |
|--------|------|-------|
| POST | `/api/v1/domains/check` | Body `{ "domainNames": ["label.kas"], "address": "kaspatest:…" }` — **both fields required** |
| GET | `/api/v1/{domain}/owner` | e.g. `kaspa.kas` → owner + assetId |
| GET | `/api/v1/assets` | Query `owner`, `type=domain`, `page`, `pageSize` |
| GET | `/api/v1/primary-name/{owner}` | 404 if none |
| GET | `/api/v1/asset/{assetId}/detail` | OpenAPI documented |

OpenAPI: extracted from `https://apidoc.knsdomains.org/tn10/swagger-ui-init.js` → saved as `openapi-tn10-real.json` / `api/openapi-v0.1.4.json` (title KNS API 0.1.4).

Public TN10 balance/UTXO: `https://api-tn10.kaspa.org/addresses/<kaspatest:…>/balance` and `…/utxos`.

Local kaspad TN10 still running (`:16211` P2P, `:16210` gRPC, `:18210` JSON wRPC). HTTP JSON curl to `:18210` returned empty reply; prefer public API or wasm RpcClient for balances.

## Protocol (from STP-KAS/kns-spec PROTOCOL.md + KasWare HTML)

- Envelope: `<xonly_pubkey> OP_CHECKSIG OP_FALSE OP_IF <kns> <0> <payload> OP_ENDIF` (commit–reveal P2SH).
- Create payload (compact JSON): `{"op":"create","p":"domain","v":"<label>"}` — `v` is label **without** `.kas`.
- Reveal **output 0** must pay KNS protocol fee address.
- TN10 fee sink (docs): `kaspatest:qq9h47etjv6x8jgcla0ecnp8mgrkfxm70ch3k60es5a50ypsf4h6sak3g0lru` (also in `KNS-TN10-PAYMENT-ADDRESS.txt`).
- Fee table (visual/grapheme length): 1–2 → 4200 KAS; 3 → 2100; 4 → 525; **5+ → 35**; text inscription → 1. Hold ~**1.05×** domain fee extra (refunded unused).
- Indexer FCFS — losing reveal still pays fee. Check before create.
- KNS does **not** support ECDSA addresses.

## KasWare notes (STP-KAS + vendor)

- STP-KAS `wallet-integration` and kns-spec `KASWARE.md` / `WALLETS.md` are **withdrawn** (17 Sep 2026 desk disclaimer). Do not treat that GitHub as a wallet kit.
- Working reference still in kns-spec HTML: `docs/kasware.html`, `docs/kasware-create.html` (mainnet-oriented copy).
- Vendor: `window.kasware.buildScript({ type: "KNS", data })` → `{ script, p2shAddress }` (no popup).
- Then `submitCommitReveal(commit, reveal, script, networkId)` with amounts in **KAS** (not sompi). Reveal requires approval.
- Network id for TN10: KasWare maps `kaspa_testnet_10` → `"testnet-10"`.
- See `docs/KASWARE-CREATE-TN10.md` for TN10-substituted HTML flow.

## Wallets / balances (public only; no seeds/keys logged)

| Role | Address file | Balance (api-tn10, sompi) | TKAS approx |
|------|----------------|---------------------------|-------------|
| File labeled “primary” / `wallet-address.txt` | `kaspatest:qplk…pv7sn` | **0** | 0 |
| Seed-derived receive0 (`seed-derived-receive-0.txt`, BIP44 via kaspa-wasm XPrivateKey) | `kaspatest:qrw0…dwrt` | 43523466024 | ~435.23 |
| Backup fund (`backup-fund-address.txt`) — **smoke payer per steering** | `kaspatest:qz46…knsdd` | 11444927204715 | ~114449 |

Parent steering: primary has 0 TKAS → use **backup funded** address for smoke. Seed-derived receive0 also holds ~435 TKAS if a later path prefers that wallet; do not print seed.

Note: BIP44 from backup **mnemonic** did not match backup fund address in an earlier desk pass; hex key matched. Prefer KasWare’s own account or verified hex for that address.

## Domain check sample (live)

`smoke-kns-tn10-001.kas` / `smoke-kns-tn10-001.kas` style names returned `available: true` via POST domains/check (with a payer address). Prefer unique labels per run (timestamp suffix).

## Blockers for smoke

1. **Payer choice**: wallet-address.txt empty; smoke must use backup (or seed-derived receive0). KasWare extension must be unlocked on that account if using UI path.
2. **No headless KasWare** on this box — browser extension required for `buildScript` / `submitCommitReveal`, **or** a custom commit–reveal builder.
3. **kaspa-wasm 0.13.0 `ScriptBuilder`** on Node returned rust null-pointer on `addData`/`addOp` in this environment — programmatic envelope build not yet reliable here; dry-run script plans txs without broadcasting.
4. kns-spec KasWare HTML examples hardcode **mainnet** fee address + `mainnet` network string — must swap for TN10 (see TN10 HTML doc).
5. Local `:18210` JSON RPC not curl-friendly; use public explorer API or wasm RPC.

## Safety

- Mainnet not used. Miner/`/tmp/kaspa-data-tn10` left running.
- Seed/private keys never written to artifacts stdout files (only addresses + public JSON).

## Next

1. Run KasWare TN10 create HTML (or dry-run script then manual broadcast gate) for one available 5+ char label using **backup** payer.
2. Confirm indexer owner via GET `/api/v1/<name>/owner`.
3. Only then consider batch creates.

## Update 2026-09-19 19:40 CEST
- computerUse first pass: Chromium failed to start on agent display (`ECONNREFUSED 127.0.0.1:9240`); no KasWare UI inspection yet.
- `box-doctor` SUMMARY: 11 checks, 0 failed; later Chrome listening on `:9240`.
- Dry-run OK: `stp-smoke-1789839537.kas` available; fee 35 TKAS; payer backup `kaspatest:qz46…knsdd` balance ~114449 TKAS. Artifact: `api/dryrun-stp-smoke-1789839537.json`.
- GitHub: https://github.com/STP-KAS/kns-kasware-tn10-test/issues/1

## Smoke result (SUCCESS) — 2026-09-19 ~19:45 CEST

Domain: `stp-smoke-1789839537.kas`
Payer: `kaspatest:qz46xwjp2stquq20xjzguqg4wyqz0g08d3clr26tlkzcrvfv0j3nxmkaknsdd`
Commit: `8f7653815d0791dd0184161a1b909d6fa4b01760880c9b750bf99fb5170f04b7`
Reveal: `04daabc03fd4ce60fcad91967a7b033539e48b1aeb1d60b02dacca4f2334bf75`
Inscription id: `04daabc03fd4ce60fcad91967a7b033539e48b1aeb1d60b02dacca4f2334bf75i0`
Indexer owner match: yes (GET `/api/v1/.../owner` returned success within poll window).

Path: scripted via kaspa-wasm32-sdk **v2.0.1** + public Resolver wRPC (`wss://vector-10.kaspa.green/...`). npm `kaspa-wasm@0.13.0` ScriptBuilder is broken (null pointer).
KasWare browser UI: still not verified (Chrome/CDP flaky on agent display).
