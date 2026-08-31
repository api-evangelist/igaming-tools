---
generated: '2026-08-30'
method: generated
name: Research a slot game and its family
description: Resolve a slot by taxonomy-correct filters, pull its full spec sheet and assets, then widen to its series and its provider.
api: openapi/igaming-tools-openapi.json
operations: [themes_list, features_list, slots_list, slots_retrieve, series_list, series_retrieve, providers_retrieve]
source: >-
  Grounded in openapi/igaming-tools-openapi.json, captured 2026-08-30 from
  https://i-gaming.tools/docs/openapi.json. Every operationId verified verbatim in that
  spec. Entity relationships per data-model/igaming-tools-data-model.yml. The same flow is
  available anonymously through the hosted MCP server — see mcp/igaming-tools-mcp.yml and
  the tool bindings in mcp/igaming-tools-tool-crosswalk.yml.
---

# Research a slot game and its family

## Auth
- REST: `Authorization: Token <your-hex-key>`, base `https://i-gaming.tools/api/v1`.
- MCP alternative: `https://mcp.i-gaming.tools/mcp`, **no key at all**, not metered. The tools `list_themes`, `list_features`, `search_slots`, `get_slot`, `list_series`, `get_series` and `get_provider` front exactly the operations below (`mcp/igaming-tools-tool-crosswalk.yml`). If you only need lookups, prefer the MCP route and spend no quota.

## Steps

1. **Resolve your taxonomy slugs first** — `themes_list` (`GET /api/v1/themes/`) and `features_list` (`GET /api/v1/features/`). Each item carries `slug`, `name`, `aliases` and `slots_count`.
2. **Search the catalog** — `slots_list` (`GET /api/v1/slots/`). Filter on `theme`, `feature`, `provider`, `series`, `volatility`, `mechanic`, `jackpot_type`, `has_bonus_buy`, `game_category`, `rtp_min`/`rtp_max`, `max_win_min`/`max_win_max`, `released_after`/`released_before`, plus free-text `search` and `ordering`.
3. **Pull the full record** — `slots_retrieve` (`GET /api/v1/slots/{slug}/`). Returns three blocks: `data` (RTP and its variants, volatility, max win, reels x rows, bet mechanic, min/max bet), `spec_sheet`, and `assets` (screenshots and derivatives).
4. **Widen to the family** — `series_list` (`GET /api/v1/series/`), optionally filtered by `provider`, then `series_retrieve` (`GET /api/v1/series/{slug}/`) for the family summary: RTP and max-win ranges, release years, roster. If `slots_truncated` is true, follow `slots_url` for the complete roster.
5. **Widen to the studio** — `providers_retrieve` (`GET /api/v1/providers/{slug}/`) for firmographics, licensing, offices, counts and HATEOAS links.

## Rules an agent must follow

- **An alias is not a filter value.** The provider states this explicitly for themes and features: look the canonical `slug` up via `themes_list` / `features_list` before filtering. Passing an alias yields `400` with code `invalid_parameter`, not an empty result — so treat a `400` here as "wrong slug", not "no matches".
- **A slot's themes and features are not returned as id arrays** on the list item. Resolve membership by querying `slots_list` with the taxonomy filter, not by reading a field (`data-model/igaming-tools-data-model.yml`).
- **No value in this dataset is inferred.** The provider's stated rule is that a value that cannot be read directly from the paytable or demo is rendered as an em dash rather than guessed. Do not fill a gap you find here with a number from elsewhere and present it as this provider's data.
- **`rtp_default` is one of up to 30 RTP variants.** Bonus-buy, ante-bet, feature and operator-configurable values differ. Never quote a single RTP figure for a game without saying which variant it is.
- **Assets are versioned for cache-busting.** Display from `url`; re-fetch only when the `version` field changes.
- **304 is your friend.** Send `If-None-Match` on repeat detail fetches — it does not decrement the monthly quota.
