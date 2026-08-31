---
generated: '2026-08-30'
method: generated
name: Analyse search demand for a market, a provider or a title
description: Read the computed demand layer three ways — one country's ranked cut, one provider's snapshot, one slot's Country x Month matrix — and report it with its staleness stamp.
api: openapi/igaming-tools-openapi.json
operations: [demand_list, providers_demand_retrieve, slots_demand_retrieve, providers_list, slots_list]
source: >-
  Grounded in openapi/igaming-tools-openapi.json, captured 2026-08-30 from
  https://i-gaming.tools/docs/openapi.json. Every operationId verified verbatim in that
  spec. Coverage caveats read from the live MCP tool descriptions in
  mcp/igaming-tools-mcp-tools.json.
---

# Analyse search demand for a market, a provider or a title

Demand is a **computed layer** sitting beside the catalog, not a field on the catalog entities. Every payload carries `computed_at`; carry it through to whatever you report.

## Auth
- REST: `Authorization: Token <your-hex-key>`, base `https://i-gaming.tools/api/v1`.
- MCP alternative (no key, not metered): tools `get_demand_cut`, `get_provider_demand`, `get_slot_demand`.

## Steps

1. **Market cut** — `demand_list` (`GET /api/v1/demand/`). A **country is required per call**. Returns the slots searched in that market ranked by 12-month volume, with a `by` dimension selector and the same structural filters as the slot search (`volatility`, `mechanic`, `jackpot_type`, `has_bonus_buy`, `provider`, `theme`, `feature`, `game_category`, `rtp_min`/`rtp_max`, release window).
2. **Provider snapshot** — `providers_demand_retrieve` (`GET /api/v1/providers/{slug}/demand/`). Returns `computed_at`, `metrics` and `top_slots`. Resolve the slug first with `providers_list` if you only have a brand name.
3. **Title trend** — `slots_demand_retrieve` (`GET /api/v1/slots/{slug}/demand/`). Returns the aggregate (`volume_12m`, `prev_12m`, `yoy_pct`, `trend`, `sparkline`) plus a full Country x Month `markets` matrix. Resolve the slug first with `slots_list`.
4. **Compare** by running step 2 for each provider in your set, or step 1 for each market, and normalising on `volume_12m` — the one measure all three shapes share.

## Rules an agent must follow

- **An empty demand payload is a real answer.** The provider's own tool description notes a demand snapshot can come back empty. That means "not computed for this entity", not zero demand. Report it as absent, never as a zero.
- **Coverage is the catalogue, not the market.** `get_demand_cut` is documented as "our catalogue only". A slot missing from a country's cut may simply not be in the catalog. Never present a cut as total market share.
- **Always surface `computed_at`.** These are snapshots. A demand figure quoted without its computation date is a claim you cannot defend.
- **`trend` is an enum, not a number.** Read `trend` for direction and `yoy_pct` for magnitude; do not derive one from the other.
- **Budget the fan-out.** Comparing twenty providers is twenty quota-decrementing calls against a 10,000/month free tier at 100 requests/minute. If the job is pure lookup, run it through the unmetered MCP server instead (`mcp/igaming-tools-mcp.yml`).
- **`402` is not `429`.** Quota exhaustion is not retryable; the per-minute throttle is. See `errors/igaming-tools-problem-types.yml`.
