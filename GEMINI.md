# TariffsAPI tools

You have the TariffsAPI MCP server: the tariff and HTS source for AI agents. All tools are read-only lookups.

- `ResolveTariffTool`(hts, origin, [as_of], [include_unknown_effective_dates]): duties for a US HTS code and an ISO-2 origin country, broken down into base rate and additional measures (e.g. Section 301, 232).
- `GetByHtsTool`(hts_number, [origin], [as_of]): details for an HTS code.
- `SearchProductsTool`(query, [limit ≤100], [has_tariffs]): search the HTS schedule by keyword.
- `ListAgreementsTool`([code]): trade agreements.
- `BatchResolveTariffsTool`(requests: up to 200 {hts, origin}, [as_of], [include_unknown_effective_dates]): many lookups in one call.

Guidance:
- TariffsAPI does not classify products. If the user has no HTS code, use SearchProductsTool to find candidates and tell the user to confirm the classification.
- Only state rates returned by the tools in this session; cite the HTS code, origin and date used.
- Use BatchResolveTariffsTool for more than a few lookups.
- Errors: 401 = missing/invalid key; 403 PLAN_UPGRADE_REQUIRED = plan lacks MCP (Developer or higher needed); 429 RATE_LIMITED (500 requests/min per key, honour Retry-After) or MONTHLY_LIMIT_EXCEEDED.
