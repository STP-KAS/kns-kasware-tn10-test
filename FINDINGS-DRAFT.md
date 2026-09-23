# kns / KasWare TN10 test findings (draft)

Date: 2026-09-19 (Europe/Brussels / CEST)

Scope: **testnet-10 only**. Do not touch mainnet. Do not stop miner farm on `:16211` or `/tmp/kaspa-data-tn10`.

The 23 Sep 2026 recheck is at the bottom. It supersedes the status lines above it. The 19 Sep sections stay as the draft they were.

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
- TN10 fee sink is documented by KNS. 5+ character names → 35 TKAS. Hold ~**1.05×** domain fee extra (refunded unused).
- Fee table (visual/grapheme length): 1–2 → 4200 KAS; 3 → 2100; 4 → 525; **5+ → 35**; text inscription → 1.
- Indexer FCFS — losing reveal still pays fee. Check before create.
- KNS does **not** support ECDSA addresses.

## KasWare notes (STP-KAS + vendor)

- STP-KAS `wallet-integration` and kns-spec `KASWARE.md` / `WALLETS.md` are **withdrawn** (17 Sep 2026 desk disclaimer). Do not treat that GitHub as a wallet kit.
- Working reference still in kns-spec HTML: `docs/kasware.html`, `docs/kasware-create.html` (mainnet-oriented copy).
- Vendor: `window.kasware.buildScript({ type: "KNS", data })` → `{ script, p2shAddress }` (no popup).
- Then `submitCommitReveal(commit, reveal, script, networkId)` with amounts in **KAS** (not sompi). Reveal requires approval.
- Network id for TN10: KasWare maps `kaspa_testnet_10` → `"testnet-10"`.
- See `docs/KASWARE-CREATE-TN10.md` for TN10-substituted HTML flow.

## Wallets / balances (roles only)

| Role | 19 Sep note |
|------|-------------|
| File labeled “primary” | **0** TKAS |
| Seed-derived receive0 | funded, smaller than the backup |
| Backup fund | **smoke payer**. BIP44 from the backup mnemonic did not match this address; a hex key did. Keep the hex out of git and issues |

Parent steering: primary had 0 TKAS, so smoke used the backup fund. KasWare extension must be unlocked on that account if using the UI path.

## Domain check sample (live)

`smoke-kns-tn10-001.kas` style names returned `available: true` via POST domains/check (with a payer address). Prefer unique labels per run (timestamp suffix).

## Blockers for smoke

1. **Payer choice**: primary empty; smoke used the backup fund. KasWare extension must be unlocked on that account if using the UI path.
2. **No headless KasWare** on this box — browser extension required for `buildScript` / `submitCommitReveal`, **or** a custom commit–reveal builder.
3. **kaspa-wasm 0.13.0 `ScriptBuilder`** on Node returned a rust null-pointer on `addData`/`addOp` in this environment.
4. kns-spec KasWare HTML examples hardcode **mainnet** fee address + `mainnet` network string — must swap for TN10.
5. Local `:18210` JSON RPC was not curl-friendly; use the public explorer API or wasm RPC.

## Safety

- Mainnet not used. Miner `/tmp/kaspa-data-tn10` was left running by the bot on 19 Sep.
- Seed and private keys were not written into artifacts.

## Smoke result (SUCCESS) — 2026-09-19 ~19:45 CEST

Domain: `stp-smoke-1789839537.kas`

Commit: `8f7653815d0791dd0184161a1b909d6fa4b01760880c9b750bf99fb5170f04b7`

Reveal: `04daabc03fd4ce60fcad91967a7b033539e48b1aeb1d60b02dacca4f2334bf75`

Inscription id: `04daabc03fd4ce60fcad91967a7b033539e48b1aeb1d60b02dacca4f2334bf75i0`

Indexer owner match: yes.

Path: scripted via kaspa-wasm32-sdk **v2.0.1** + public Resolver wRPC. npm `kaspa-wasm@0.13.0` ScriptBuilder is broken (null pointer).

KasWare browser UI: not verified. The extension was not installed.

## Recheck — 23 Sep 2026 (Grok Build, Windows desk)

This section is the current status. The bot paused on 20 Sep 2026.

| Check | Result |
| --- | --- |
| `stp-bulk-0001.kas` through `stp-bulk-0750.kas` | **750/750** owner reads on `https://api.knsdomains.org/tn10` |
| `0751` through `0760` | **404** |
| Former holes `0056`, `0058`, `0134`, `0139` | Present |
| Smoke `stp-smoke-1789839537.kas` | Still indexed |
| Pause | Grok Bot weekly pool, Thu 24 Sep 2026 ~17:44 Europe/Brussels. The next issue comment says 17:50. Meter not re-read here |
| Resume `0751`→`3000` | Named on the pause comment. **No resume script is in this git tree.** Not started |
| `inscriptions-digest.md` / `.csv` | 19 Sep snapshot, 58 rows. Markdown skips `0056` and `0058`. Not the 750 record |
| Bulk script | Not in git |

This repo is the tn10 Grok Bot lab. The Grok Bot weekly pool is not the grok.com Settings → Usage bar.

Uniqueness of these names is indexer FCFS. The creates are pubkey spends with a `kns` envelope. They are not Toccata covenant programs. `covenant_id` does not encode the label.

Issue: https://github.com/STP-KAS/kns-kasware-tn10-test/issues/1
