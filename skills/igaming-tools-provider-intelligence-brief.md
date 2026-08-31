---
generated: '2026-08-30'
method: generated
name: Build an intelligence brief on a slot studio
description: Assemble one provider's profile, catalog, hiring, news coverage, key people and licensing regulators into a single sourced brief.
api: openapi/igaming-tools-openapi.json
operations: [providers_list, providers_retrieve, providers_slots_list, providers_news_list, providers_jobs_list, providers_team_list, regulators_list, regulators_retrieve, sources_list]
source: >-
  Grounded in openapi/igaming-tools-openapi.json, captured 2026-08-30 from
  https://i-gaming.tools/docs/openapi.json. Every operationId verified verbatim in that
  spec. Entity relationships per data-model/igaming-tools-data-model.yml.
---

# Build an intelligence brief on a slot studio

## Auth
- `Authorization: Token <your-hex-key>`, base `https://i-gaming.tools/api/v1`.
- Note: four of the nine operations below (`providers_news_list`, `providers_jobs_list`, `providers_team_list`, `sources_list` detail) have **no MCP equivalent** — this brief is a REST-only flow (`mcp/igaming-tools-tool-crosswalk.yml`).

## Steps

1. **Resolve the studio** — `providers_list` (`GET /api/v1/providers/`) with `query`, `country`, `established_after` / `established_before`, or `has_news`. Take the `slug`.
2. **Profile** — `providers_retrieve` (`GET /api/v1/providers/{slug}/`): `tagline`, `about_html`, `firmographics`, `company`, `counts`, `hosts`, `links`, plus `verified_at` and `updated_at`.
3. **Catalog** — `providers_slots_list` (`GET /api/v1/providers/{slug}/slots/`) with structural filters (`volatility`, `mechanic`, `jackpot_type`, `has_bonus_buy`, `theme`, `feature`).
4. **Coverage** — `providers_news_list` (`GET /api/v1/providers/{slug}/news/`) for articles attributed to the brand. Cross-reference the publishing hosts with `sources_list` (`GET /api/v1/sources/`) to see who is actually writing about them and how active each host is (`articles_count_30d`).
5. **Hiring** — `providers_jobs_list` (`GET /api/v1/providers/{slug}/jobs/`). Vacancy records carry `igaming_segment`, `seniority`, `remote_mode`, `employment_type`, salary band, and `status_public` / `closed_at` / `closed_reason`.
6. **People** — `providers_team_list` (`GET /api/v1/providers/{slug}/team/`) for key team members.
7. **Regulators** — `regulators_list` (`GET /api/v1/regulators/`) and `regulators_retrieve` (`GET /api/v1/regulators/{slug}/`) for jurisdiction, `authority_for` and `statutory_authority` on the authorities relevant to the studio's licensing.

## Rules an agent must follow

- **Providers and regulators are the same underlying entity.** Both are "brands" discriminated by a `kind` field, which is why `providers_news_list` and `regulators_news_list` return identical shapes. Do not treat a regulator record as a company record (`data-model/igaming-tools-data-model.yml`).
- **`company_name` and `company` are not the same field on a job.** `company_name` is the raw string from the source posting; `company` is the resolved brand slug and may be null. Attribute on `company`, and say "unresolved" when it is null rather than string-matching your way to a guess.
- **Closed vacancies are excluded by default.** Pass `include_closed` deliberately if you are measuring hiring over time, and read `closed_reason` before inferring anything from a closure.
- **News is ingested from third-party hosts, not authored here.** `NewsArticleFull.body_unavailable_reason` exists precisely because a body sometimes cannot be retrieved. Cite the original `url` and `host`, never present the article as the provider's own reporting.
- **Carry the freshness stamps.** `verified_at`, `updated_at`, `last_extracted_at`, `discovered_at` and `last_seen_at` all appear in these payloads. A brief without them is undated.
- **Budget it.** A full brief on one studio is 7-plus quota-decrementing calls. `304` responses cost no quota — cache ETags between runs (`rate-limits/igaming-tools-rate-limits.yml`).
