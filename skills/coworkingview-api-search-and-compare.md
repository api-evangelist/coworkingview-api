---
name: coworkingview-search-and-compare
description: Find and compare coworking spaces in a CoworkingView-covered city using the read-only /v1 REST API.
api: CoworkingView REST API
base_url: https://api.coworkingview.com
operations:
  - geoBootstrap
  - searchSuggest
  - facets
  - listProperties
  - propertyDetail
  - marketRates
  - rankings
---

# Search and compare coworking spaces

Read-only, no API key. All calls take a required `locale` (`en`, `de`, or `es`). Slugs are lowercase; a guessed
slug returns an empty result, not an error — always resolve a place first.

## Steps

1. **Resolve the place.** `GET /v1/geo/bootstrap?locale=en` (operationId `geoBootstrap`) to get the country/city/zone
   slug tree with space counts, or `GET /v1/search/suggest?locale=en&q=...` (`searchSuggest`) to resolve free text
   into canonical slugs.
2. **(Optional) Discover filters.** `GET /v1/facets?locale=en&city={city}` (`facets`) to learn the amenities,
   workspace types and price bands available in that scope.
3. **Search.** `GET /v1/properties?locale=en&city={city}` (`listProperties`) with any of `zone`, `operator`,
   `amenities`, `minCapacity`, `minPrice`, `maxPrice`, `currency`, `pricingOptionType`, geo (`lat`/`lng`/`radiusMeters`
   or `bbox`), `sortBy`. Page with `page` + `pageSize`; pass the returned `seed` on later pages to keep ordering stable.
4. **Detail.** `GET /v1/properties/{slug}` (`propertyDetail`) for full address, capacity, amenities and
   operator-published pricing on a shortlist.
5. **Context.** `GET /v1/market/rates?locale=en&city={city}` (`marketRates`) for the city price index, and
   `GET /v1/rankings?locale=en&city={city}` (`rankings`) for a ranked shortlist.

## Rules

- Do not invent availability or quote a price CoworkingView did not publish. Prices are in each space's own currency.
- Errors are RFC 9457 `application/problem+json` with an enumerated `code` and a `traceId`; `404 NOT_FOUND` means the
  slug does not exist (resolve it first), `429 RATE_LIMITED` means back off (limits are per IP).
