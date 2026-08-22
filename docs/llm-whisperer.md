# LLM Whisperer Console — clinicians talk to the quantum evidence

**Status:** live · **Console:** `public/whisperer.html` (deployed at
`arunnadarasa.github.io/nhscep2026/whisperer.html`)
**Bridge:** `tools/llm_whisperer_bridge.py` (stdlib HTTP, port 8788)

---

## What it is

The **H2A (human-to-agent)** layer of the Clinical Agentic Ecosystem: a clinician
types a plain-English question and the console routes it through the **same MCP
tool handlers** that serve other agents (the EndoTrack MCP server). No code, no
query language — but *every* answer comes from committed, verified JSON.

```
Clinician:  "what's the certified AUC on the order-3 label?"
   │  GET/POST /ask {"question": …}
   ▼
Bridge (tools/llm_whisperer_bridge.py)  ── intent router ──►  get_certified_results
   │                                                          (the MCP handler)
   ▼
Response: "order-3 L3 zz=0.05: quantum AUC 0.480 ± 0.118 vs classical best 0.585 (n=56) — classical wins; the n=28 edge is withdrawn (did not replicate)".
 ```

## Run it (two terminals)

```bash
# 1. start the bridge (routes intents through the MCP handlers)
python3 tools/llm_whisperer_bridge.py 8788
#    -> EndoTrack LLM-whisperer bridge on http://localhost:8788

# 2. open the console
open public/whisperer.html      # or the deployed portal copy:
                                # https://arunnadarasa.github.io/nhscep2026/whisperer.html
```

Type a question or tap a suggested chip. The console POSTs to the bridge URL in
the field (default `http://localhost:8788`, CORS-enabled for local demo).

## Supported intents (deterministic router — or Hermes itself)

The bridge routes through the MCP handlers. By default the router is
**deterministic keyword matching** (zero API keys, instant). Toggle
**🧠 Hermes LLM routing** in the console (or start the bridge with
`HERMES_CLASSIFY=1`) and every question is classified by **Hermes itself**
(`hermes -z`), so free-form phrasing works too — the response reports
`router: hermes-llm` vs `router: deterministic`. Deterministic fallback keeps
the demo alive if Hermes is slow/unavailable. The MCP handlers behind both are
identical.

### Embedded paid flow (autopay)

The console's **💸 paid evidence query** bar runs the full x402 loop in one
click: 402 challenge → settlement → verified receipt + artifact, in
**USDC (native) · EURC · cirBTC**. By default (`X402_MOCK=1`) settlement is a
mock demo tx; with `X402_MOCK=0` + `ARC_RPC_URL` + `X402_PAYER_KEY` the bar
**broadcasts a real Arc testnet payment** (proven 2026-08-20: EURC and USDC
console-minted txs, Arcscan-verified — see `docs/x402-demo.md`).

| Question pattern | MCP tool | Sample answer |
|---|---|---|
| "AUC / order-3 / order-1 / certified" | `get_certified_results` | 0.480 ± 0.118 vs 0.585 (n=56) — classical wins; n=28 edge withdrawn |
| "verify <file>.json" | `verify_artifact` | Verified ✓ — committed JSON, N bytes |
| "triage / cells / 16/16 / sweep / pass" | `get_triage_rows` | 16 cells, n 4→10, all dist_pass=True |
| "playpond / demo / share links" | `get_playpond_links` | 9 verified one-click browser demos |
| "roadmap / phase / when" | `get_roadmap_status` | 8 phases, hardware-indexed |
| "model card / transparency / rri" | `get_model_card` | 11 sections, docs/quantum-card.md |
| "paid / price / usdc / eurc / x402" | `verify_artifact_paid` | $0.01 USDC on Arc 5042002 → receipt |

The router is deterministic keyword matching so the demo runs with **zero API
keys**; the intent→tool mapping is the only LLM-substitutable part — the MCP
handlers behind it are the same ones other agents call.

## Why it matters for top-4

- **The "LLM whisperer" thesis**: clinicians don't code, but they *can* direct
  agents. The console is that interface — H2A in one screen, backed by the same
  certified pipeline the A2A agents use (MCP) and the same payments (x402).
- **Evidence discipline survives translation**: the answer strings always cite
  the committed artifact (`results/…json`), because the values come *from* it.
- **Demo magic**: run the bridge, open the console, and let a judge ask
  "what's the certified AUC?" — the answer appears with the tool name and its
  source file, proving the whole stack (portal → bridge → MCP → JSON) is real.

## Honest limitations

- The bridge is a local HTTP server — the deployed `whisperer.html` page needs
  the bridge reachable (localhost for demos). A hosted bridge (or an LLM
  classifier in `route()`) is the production step.
- CORS is open (`*`) for the demo; bind to localhost/authenticate before
  exposing beyond the hackathon.
- Deterministic router answers pattern-matched questions; an LLM classifier
  (Hermes itself) would handle free-form phrasing — the handlers are unchanged.

## Files

- `tools/llm_whisperer_bridge.py` — stdlib HTTP bridge + intent router (7 routes)
- `public/whisperer.html` — the console (also deployed to the portal)
- `tools/endo_mcp_server.py` — the MCP handlers the bridge routes to
- `docs/a2a-agent-card.md` — the A2A sibling: agents discover/delegate the same skills