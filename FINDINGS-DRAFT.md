# kns / KasWare TN10 test findings (draft)

Date: 2026-09-19 (Europe/Brussels)

## Scope
Testnet-10 only. Site: https://tn10.knsdomains.org/  
Indexer: https://api.knsdomains.org/tn10/api/v1  
OpenAPI UI: https://apidoc.knsdomains.org/tn10/

## What works
- `GET /api/v1/{domain}/owner` — resolves (e.g. `test.kas`).
- `GET /api/v1/assets?owner=...` — lists assets for an owner.
- `POST /api/v1/domains/check` with body `{ "domainNames": [...], "address": "kaspatest:..." }` — availability + reserved flags. Max 100 names/request.
- Smoke name `smoke-kns-tn10-001.kas` was **available** at check time.
- Create inscription envelope documented: commit/reveal P2SH, protocol id `kns`, domain JSON `{op:"create",p:"domain",v:"<label>"}`.
- TN10 KNS fee sink (docs): `kaspatest:qq9h47etjv6x8jgcla0ecnp8mgrkfxm70ch3k60es5a50ypsf4h6sak3g0lru`
- KasWare path (from STP-KAS/kns-spec HTML notes): `window.kasware.buildScript({type:"KNS", data})` then `submitCommitReveal` (amounts in KAS). Extension required.
- Public balance API: `https://api-tn10.kaspa.org/addresses/<kaspatest:...>/balance`

## Bugs / friction
- npm package `kaspa@0.13.0` is broken without vendored wasm (`Cannot find module './kaspa/kaspa_wasm'`). Use `kaspa-wasm@0.13.0` directly.
- `RpcClient` against local `17210` OOM/panic without proper WebSocket shim + newer SDK (groks-wallet uses a Windows-local wasm SDK path).
- Local TN10 node IBD incomplete (~33% bodies) but tip-chase; prefer `api-tn10.kaspa.org` for balances when unsure.
- STP-KAS/kns-spec `KASWARE.md` / `WALLETS.md` intentionally withdrawn (desk disclaimer) — use upstream KNS gitbook + KasWare docs.
- X MCP: `get_usage_credits` / tweet fetch blocked (`user-not-enrolled` Pay-per-use). Tweet https://x.com/knsdomains/status/2101290557821899234 not retrieved yet.
- BIP44 derive from backup **mnemonic** via `m/44'/111111'/0'/0/0` did **not** match user-provided funding address; **hex private key** did match. Treat hex (or KasWare’s own path) as source of truth for that address.

## API limits (observed / documented)
- `domains/check`: max **100** domains per request; `address` required.
- `assets` pageSize max **100**.
- No bulk-create HTTP API — inscriptions are L1 commit/reveal; indexer only reads/checks.

## Wallet notes
- Primary KNS test mnemonic: env `KNS_TN10_SEED` (secure). Derived receive (wasm BIP44): see `wallet-address.txt`.
- Backup funding address: `kaspatest:qz46xwjp2stquq20xjzguqg4wyqz0g08d3clr26tlkzcrvfv0j3nxmkaknsdd` (hex + mnemonic stored under `/home/box/.secrets/`, never in chat).

## Next
1. Confirm primary balance; fund from backup if needed.
2. KasWare in box browser OR programmatic commit/reveal smoke for one available name.
3. Open GitHub issue on STP-KAS (new `kns-kasware-tn10-test` or existing `kns` / `tn10-hard-test`) with this report.
4. Pace toward ~3000 creates only after smoke OK (check ≤100/batch, pay fees, rate-limit).
