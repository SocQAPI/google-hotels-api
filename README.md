<!-- Generated from the SocQ API catalog and docs. Do not edit by hand. -->
# Google Hotels API

[![SocQ Google Hotels API](https://img.shields.io/badge/API-SocQ%20Google%20Hotels%20API-F64C31)](https://socq.ai/apis/google-hotels?utm_source=github&utm_medium=readme&utm_campaign=socq-google-hotels-api)
[![API documentation](https://img.shields.io/badge/Docs-docs.socq.ai-111827)](https://docs.socq.ai/api-manual/google-hotels/overview)
[![License: MIT](https://img.shields.io/badge/License-MIT-2563EB)](LICENSE)
[![Check examples](https://github.com/SocQAPI/google-hotels-api/actions/workflows/check.yml/badge.svg)](https://github.com/SocQAPI/google-hotels-api/actions/workflows/check.yml)

3 endpoints for public Google Hotels data through one SocQ API key, with runnable cURL, Node.js and Python examples and a valid request body for every endpoint.

[Browse Google Hotels APIs](https://socq.ai/apis/google-hotels?utm_source=github&utm_medium=readme&utm_campaign=socq-google-hotels-api) · [Get an API key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-google-hotels-api) · [Documentation](https://docs.socq.ai/api-manual/google-hotels/overview) · [All SocQ examples](https://github.com/SocQAPI/socq-examples)

## Endpoints

Prices are in SocQ credits; 1 credit = $0.005.

| Endpoint | publicId | What it returns | Price | Launched |
| --- | --- | --- | --- | --- |
| [Google Hotels API](#google-hotels-api) | `google-hotels/search` | Search Google Hotels for a destination with stars, guest ratings, nightly prices for your dates, location and photos. | 0.15 credits ($0.00075) per result | 2026-11-17 |
| [Google Hotel Details API](#google-hotel-details-api) | `google-hotels/hotel-details` | Get a hotel's address, amenities, location scores, rating breakdown, review topics and the offer from each booking site. | 2 credits ($0.01) per call | 2026-11-17 |
| [Hotel Price Tracker API](#hotel-price-tracker-api) | `google-hotels/hotel-prices` | Get a hotel's lowest nightly price for each check-in date over the coming weeks from Google Hotels. | 4 credits ($0.02) per call | 2026-11-17 |

## Quick start

1. Create an API key at [socq.ai/dashboard/api-key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-google-hotels-api).
2. Export it. The examples read only `SOCQ_API_KEY`.

```bash
git clone https://github.com/SocQAPI/google-hotels-api.git
cd google-hotels-api
export SOCQ_API_KEY="your-api-key"
```

3. Run any endpoint by its publicId. Without a second argument the script sends `payloads/<slug>.json`; pass a JSON file or an inline JSON object to change the input.

```bash
# Node.js 20+
node examples/node/socq.mjs google-hotels/search

# Python 3.8+ (standard library only)
python3 examples/python/socq.py google-hotels/search payloads/search.json

# cURL + jq
bash examples/curl/socq.sh google-hotels/search '{"query":"hotels in new york","currency":"USD","results_limit":18}'
```

Each script submits the task, polls it until it succeeds or fails, follows every result page, and writes the records to `output/<slug>-<task_id>.json`. Run `npm run check` to validate the payloads and scripts offline.

Never expose `SOCQ_API_KEY` in browser code, mobile apps, public repositories, screenshots, fixtures or logs.

## How the API works

Base URL `https://api.socq.ai`. Every request sends the API key as a bearer token:

```http
Authorization: Bearer <SOCQ_API_KEY>
Content-Type: application/json
```

All endpoints are asynchronous:

```text
POST /v1/<publicId>  ->  data.task_id  ->  GET /v1/tasks/{task_id}  ->  queued | running | succeeded | failed
```

1. **Submit** the JSON body to `POST https://api.socq.ai/v1/<publicId>`. The response returns `data.task_id` immediately; it means the task was created, not that data is ready.
2. **Poll** `GET https://api.socq.ai/v1/tasks/{task_id}` until `data.status` is `succeeded` or `failed` (a failed task carries `data.error_message`).
3. **Read results.** A succeeded task returns normalized records in `data.results.items`. Use `limit` (default 50, max 500) and pass `data.results.next_cursor` as `cursor` while `data.results.has_more` is true. `fields` selects up to 50 comma-separated fields or dot paths.
4. **Raw files (optional).** `GET https://api.socq.ai/v1/tasks/{task_id}/files` lists raw JSONL exports.

```bash
curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100" \
  -H "Authorization: Bearer $SOCQ_API_KEY"

curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100&cursor=$NEXT_CURSOR" \
  -H "Authorization: Bearer $SOCQ_API_KEY"
```

`callback_url` (where an endpoint accepts it) makes SocQ POST to your URL when the task reaches a terminal state; keep polling as a fallback.

**Billing.** A submit may reserve credits from `results_limit` or the number of submitted inputs; when the task finishes SocQ settles the charge from the actual result count and returns unused reserved credits. `402` means insufficient credits.

**Errors.** `400` invalid body or unsupported field, `401` missing or invalid key, `402` insufficient credits, `403` IP allowlist or task owned by another account, `404` endpoint or task not found, `422` input validation failed, `429` rate or credit-window limit (respect `Retry-After`), `500` submission failed. A client-side timeout does not cancel a task: resume polling the same `task_id` instead of submitting again.

Reference: [Authentication](https://docs.socq.ai/api-manual/agent-api/authentication) · [Task status and results](https://docs.socq.ai/api-manual/agent-api/status) · [Task files](https://docs.socq.ai/api-manual/agent-api/files) · [Callbacks](https://docs.socq.ai/api-manual/agent-api/callbacks) · [Errors](https://docs.socq.ai/api-manual/agent-api/errors)

## Endpoint reference

### Google Hotels API

Search Google Hotels for a destination with stars, guest ratings, nightly prices for your dates, location and photos.

- **Request:** `POST https://api.socq.ai/v1/google-hotels/search`
- **publicId:** `google-hotels/search`
- **Price:** 0.15 credits ($0.00075) per result
- **Launched:** 2026-11-17
- **Example body:** [`payloads/search.json`](payloads/search.json)

Search hotels the way travelers do on Google, then work with the list as data. Each hotel comes with its name, star class, guest rating and review count, the lowest nightly price for your dates, map coordinates and photos, plus the `id` you pass to Hotel Details and Hotel Prices. Add check-in and check-out dates and a currency to price a real trip, sort by price or rating, and narrow the list with a price range.

Provide a destination or hotel search in query. check_in and check_out (YYYY-MM-DD, given together, up to 30 nights) price a specific stay; without them prices are for one night from tomorrow. sort_by accepts relevance, lowest_price, highest_rating or most_reviewed. results_limit defaults to 18 and accepts integers from 1 to 140. Each hotel costs 0.15 credits.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `check_in` | string | No | Check-in date in YYYY-MM-DD format. Defaults to tomorrow; give check_out with it. (pattern `^[0-9]{4}-[0-9]{2}-[0-9]{2}$`) |
| `check_out` | string | No | Check-out date in YYYY-MM-DD format, after check_in and at most 30 nights later. (pattern `^[0-9]{4}-[0-9]{2}-[0-9]{2}$`) |
| `currency` | string | No | Three-letter currency code for prices, such as USD or EUR. (pattern `^[A-Za-z]{3}$`) |
| `language_code` | string | No | Language code of the results. Defaults to en. (length 2 to 10; default `"en"`) |
| `location_code` | integer | No | Search market location code. Defaults to 2840 (United States). (value at least 1; default `2840`) |
| `max_price` | number | No | Highest nightly price to include, in the chosen currency. (value at least 0) |
| `min_price` | number | No | Lowest nightly price to include, in the chosen currency. (value at least 0) |
| `query` | string | Yes | Destination or hotel search, such as `hotels in new york`. |
| `results_limit` | integer | No | (value 1 to 140; default `18`) |
| `sort_by` | string | No | (one of `highest_rating`, `lowest_price`, `most_reviewed`, `relevance`) |

```bash
curl -X POST "https://api.socq.ai/v1/google-hotels/search" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "query": "hotels in new york",
  "currency": "USD",
  "results_limit": 18
}'
```

Full docs: [https://docs.socq.ai/api-manual/google-hotels/search](https://docs.socq.ai/api-manual/google-hotels/search) · Landing page: [socq.ai/apis/google-hotels/search](https://socq.ai/apis/google-hotels/search?utm_source=github&utm_medium=readme&utm_campaign=socq-google-hotels-api)

### Google Hotel Details API

Get a hotel's address, amenities, location scores, rating breakdown, review topics and the offer from each booking site.

- **Request:** `POST https://api.socq.ai/v1/google-hotels/hotel-details`
- **publicId:** `google-hotels/hotel-details`
- **Price:** 2 credits ($0.01) per call
- **Launched:** 2026-11-17
- **Example body:** [`payloads/hotel-details.json`](payloads/hotel-details.json)

Everything Google Hotels knows about one property in a single row. You get the address, phone and website, check-in time, the amenity list, scores for the neighborhood's transit, restaurants and attractions, the share of reviews at each star rating, the topics guests praise or complain about, ratings on other review sites, and the price each booking site charges for your stay. Ideal for rate-parity checks, competitive set analysis or a richer hotel page in your own product.

Provide a hotel_id from the id field of Google Hotels Search. Add check_in and check_out (YYYY-MM-DD) to get offers for those dates; otherwise offers are for one night from tomorrow. Each request costs 2 credits and returns one row; an unknown hotel_id returns no rows and is still charged.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `check_in` | string | No | Check-in date in YYYY-MM-DD format. Defaults to tomorrow; give check_out with it. (pattern `^[0-9]{4}-[0-9]{2}-[0-9]{2}$`) |
| `check_out` | string | No | Check-out date in YYYY-MM-DD format, after check_in and at most 30 nights later. (pattern `^[0-9]{4}-[0-9]{2}-[0-9]{2}$`) |
| `currency` | string | No | Three-letter currency code for prices, such as USD or EUR. (pattern `^[A-Za-z]{3}$`) |
| `hotel_id` | string | Yes | Hotel ID from the `id` field of Google Hotels Search results. |
| `language_code` | string | No | Language code of the results. Defaults to en. (length 2 to 10; default `"en"`) |
| `location_code` | integer | No | Search market location code. Defaults to 2840 (United States). (value at least 1; default `2840`) |

```bash
curl -X POST "https://api.socq.ai/v1/google-hotels/hotel-details" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "hotel_id": "ChoIyKbJ6puqvPf8ARoNL2cvMTFqNHR2MjczeBAB",
  "currency": "USD"
}'
```

Full docs: [https://docs.socq.ai/api-manual/google-hotels/hotel-details](https://docs.socq.ai/api-manual/google-hotels/hotel-details) · Landing page: [socq.ai/apis/google-hotels/hotel-details](https://socq.ai/apis/google-hotels/hotel-details?utm_source=github&utm_medium=readme&utm_campaign=socq-google-hotels-api)

### Hotel Price Tracker API

Get a hotel's lowest nightly price for each check-in date over the coming weeks from Google Hotels.

- **Request:** `POST https://api.socq.ai/v1/google-hotels/hotel-prices`
- **publicId:** `google-hotels/hotel-prices`
- **Price:** 4 credits ($0.02) per call
- **Launched:** 2026-11-17
- **Example body:** [`payloads/hotel-prices.json`](payloads/hotel-prices.json)

The price calendar behind Google Hotels' date picker, as rows. For one hotel you get the lowest price for each check-in night, usually six to eight weeks of dates starting from your check-in. Spot the cheapest nights to travel, chart how a hotel prices weekends against weekdays, or feed a revenue manager's rate-shopping report.

Provide a hotel_id from the id field of Google Hotels Search. check_in sets the first date of the calendar (tomorrow when omitted); give check_out with it. Each request costs 4 credits; each row is one check-in date with the price for one night.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `check_in` | string | No | Check-in date in YYYY-MM-DD format. Defaults to tomorrow; give check_out with it. (pattern `^[0-9]{4}-[0-9]{2}-[0-9]{2}$`) |
| `check_out` | string | No | Check-out date in YYYY-MM-DD format, after check_in and at most 30 nights later. (pattern `^[0-9]{4}-[0-9]{2}-[0-9]{2}$`) |
| `currency` | string | No | Three-letter currency code for prices, such as USD or EUR. (pattern `^[A-Za-z]{3}$`) |
| `hotel_id` | string | Yes | Hotel ID from the `id` field of Google Hotels Search results. |
| `language_code` | string | No | Language code of the results. Defaults to en. (length 2 to 10; default `"en"`) |
| `location_code` | integer | No | Search market location code. Defaults to 2840 (United States). (value at least 1; default `2840`) |

```bash
curl -X POST "https://api.socq.ai/v1/google-hotels/hotel-prices" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "hotel_id": "ChoIyKbJ6puqvPf8ARoNL2cvMTFqNHR2MjczeBAB",
  "currency": "USD"
}'
```

Full docs: [https://docs.socq.ai/api-manual/google-hotels/hotel-prices](https://docs.socq.ai/api-manual/google-hotels/hotel-prices) · Landing page: [socq.ai/apis/google-hotels/hotel-prices](https://socq.ai/apis/google-hotels/hotel-prices?utm_source=github&utm_medium=readme&utm_campaign=socq-google-hotels-api)

## Repository contents

| Path | Purpose |
| --- | --- |
| [`examples/curl/socq.sh`](examples/curl/socq.sh) | Submit, poll and paginate with cURL and jq |
| [`examples/node/socq.mjs`](examples/node/socq.mjs) | Same workflow in Node.js 20+ (no dependencies) |
| [`examples/python/socq.py`](examples/python/socq.py) | Same workflow in Python 3.8+ (standard library) |
| [`payloads/`](payloads) | One documented request body per endpoint |
| [`endpoints.json`](endpoints.json) | Endpoint list, prices and request schemas used by the scripts and checks |
| [`fixtures/`](fixtures) | Synthetic submit and task response shapes |
| [`docs/production-notes.md`](docs/production-notes.md) | Retries, timeouts, pagination and data handling |
| [`scripts/check.mjs`](scripts/check.mjs) | Offline check: payloads, syntax and secrets |

## Responsible use and platform scope

- Use only publicly accessible Google Hotels data and the fields supported by each endpoint.
- Do not use the examples to access restricted content, login-only surfaces, authentication controls or non-public personal information.
- SocQ is not an official API of the represented platform and is not affiliated with or endorsed by it. Platform names and trademarks belong to their respective owners.
- Before production use, assess the laws, platform terms, privacy obligations and retention requirements that apply to your organization and use case.
- Collect only the fields needed for a defined purpose, restrict access, set retention periods, and support correction or deletion workflows where required.

This section describes the public-data boundary; it is not legal advice.

## License

[MIT](LICENSE)
