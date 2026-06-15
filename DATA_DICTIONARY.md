# Data Dictionary

Field-level documentation for all JSON schemas in this dataset.

---

## 1. Crawl Log Entries (`crawl-log_*.json`)

Top-level structure:

| Field | Type | Description |
|-------|------|-------------|
| `date` | string | Date of the log in YYYYMMDD format. |
| `count` | integer | Total number of entries in this log. |
| `crawlerOnly` | boolean | If true, only crawler-identified entries are included (filtered view). |
| `entries` | array | Array of individual HTTP request records. |

### Entry fields (`entries[*]`):

| Field | Type | Description |
|-------|------|-------------|
| `ts` | string (ISO 8601) | Timestamp of the HTTP request in UTC. |
| `path` | string | Requested URL path (e.g., `/robots.txt`, `/paper6.html`). |
| `method` | string | HTTP method (GET, POST, HEAD, etc.). |
| `ua` | string | Full User-Agent header string as declared by the client. |
| `ip_hash` | string (16 hex chars) | SHA-256 hash of the client IP address, truncated to 16 characters. Used for session linking without exposing raw IPs. |
| `country` | string (ISO 3166-1 alpha-2) | Country code derived from Cloudflare's `cf-ipcountry` header. |
| `asn` | integer | Autonomous System Number of the client IP. |
| `asOrg` | string | Organization name associated with the ASN (e.g., "Microsoft Corporation", "Google LLC"). |
| `crawler` | object | Crawler identification result (see below). |
| `referer` | string | HTTP Referer header value. Empty string if absent. |
| `honeypot` | boolean | True if the requested path is a honeypot (not linked from any page). |
| `pdf` | boolean | True if the request targeted a PDF file. |
| `scanner` | boolean | True if the request was identified as a security scanner or vulnerability probe (e.g., requests for `.env`, `.git/config`, `phpinfo.php`). |

### Crawler identification (`entries[*].crawler`):

| Field | Type | Description |
|-------|------|-------------|
| `detected` | boolean | True if the request was identified as a known AI/search crawler. |
| `vendor` | string or null | Identified vendor name (e.g., "OpenAI", "Google", "Anthropic", "Moz", "Microsoft", "Apple"). Null if not identified. |
| `token` | string or null | The specific crawler token matched in the User-Agent (e.g., "OAI-SearchBot", "Googlebot", "anthropic-ai"). Null if not identified. |
| `declared_purpose` | string or null | Self-declared purpose from crawler registry (e.g., "search", "seo", "training"). Null if not identified. |
| `method` | string or null | Detection method used: `"ua"` (User-Agent string match), `"rdns"` (reverse DNS verification). Null if not identified. |
| `rdns` | string or null | Reverse DNS hostname if resolved. Null if not resolved or not applicable. |
| `rdns_status` | string or null | Status of reverse DNS lookup: `"no_ptr"` (no PTR record), `"verified"`, `"mismatch"`. Null if not attempted. |

---

## 2. Canary Token Baseline Files (`canary_baseline_*.json`)

Top-level structure:

| Field | Type | Description |
|-------|------|-------------|
| `test_type` | string | Always `"canary_token_baseline"`. |
| `vendor` | string | AI vendor name (e.g., "Anthropic", "xAI"). |
| `model` | string | Model identifier used for the test (e.g., "claude-sonnet-4-6", "grok-3"). |
| `date` | string (YYYY-MM-DD) | Date the baseline was collected. |
| `settings` | string | Human-readable description of test conditions (e.g., "API direct call, no system prompt, no web retrieval"). |
| `results` | array | Array of per-token query results. |

### Result fields (`results[*]`):

| Field | Type | Description |
|-------|------|-------------|
| `token` | string | The canary token name (present in xAI file; absent in Anthropic baseline but derivable from query). |
| `query` | string | The full natural-language query sent to the model. |
| `model` | string | Model identifier for this specific query. |
| `timestamp_request` | string (ISO 8601) | Timestamp when the API request was sent. |
| `timestamp_response` | string (ISO 8601) | Timestamp when the API response was received. |
| `response_text` | string | Full text of the model's response. |
| `stop_reason` / `finish_reason` | string | Reason the model stopped generating: `"end_turn"` (Anthropic) or `"stop"` (xAI). |
| `usage` | object | Token usage counts. |
| `usage.input` | integer | Number of input tokens consumed. |
| `usage.output` | integer | Number of output tokens generated. |
| `response_id` | string | Vendor-specific unique identifier for the response (Anthropic: `msg_*`; xAI: UUID). |

---

## 3. Canary Daily Monitor Files (`canary_daily_*.json`)

Extends the baseline schema with additional fields for longitudinal monitoring.

Top-level structure includes all baseline fields plus:

| Field | Type | Description |
|-------|------|-------------|
| `test_type` | string | Always `"canary_daily_monitor"`. |
| `run_timestamp` | string (ISO 8601) | Timestamp of this monitoring run. |
| `summary` | object | Aggregate summary of results. |

### Additional result fields (`results[*]`):

| Field | Type | Description |
|-------|------|-------------|
| `token` | string | The canary token name. |
| `timestamp` | string (ISO 8601) | Timestamp of this individual query (replaces `timestamp_request`/`timestamp_response` pair). |
| `response_hash` | string (16 hex chars) | Hash of the response text for change detection across runs. |
| `is_negative` | boolean | True if the model did not recognize the canary token (expected result). False would indicate potential pre-training contamination. |

### Summary fields (`summary`):

| Field | Type | Description |
|-------|------|-------------|
| `total` | integer | Total number of tokens tested. |
| `negative` | integer | Number of tokens not recognized (expected). |
| `positive` | integer | Number of tokens recognized (unexpected; would indicate pre-training ingestion). |
| `alerts` | array | List of tokens that triggered a positive result. Empty if all negative. |
| `status_changed_from_previous` | array | List of tokens whose status changed from the previous run. Empty on first run. |

---

## 4. Aggregate Statistics Files (`stats_*.json`, `crawl-stats_full_*.json`)

Top-level structure:

| Field | Type | Description |
|-------|------|-------------|
| `days` | integer | Number of days covered by this statistics file. |
| `count` | integer | Total number of entries (present in single-day stats only). |
| `stats` | array | Array of per-day summary objects. |

### Per-day summary (`stats[*]`):

| Field | Type | Description |
|-------|------|-------------|
| `date` | string (YYYY-MM-DD) | Calendar date. |
| `total` | integer | Total HTTP requests received. |
| `crawlers` | integer | Requests identified as known AI/search crawlers. |
| `honeypot` | integer | Requests to honeypot paths. |
| `pdf` | integer | Requests for PDF files. |
| `scanners` | integer | Requests identified as security scanners or vulnerability probes. |
| `legitimate` | integer | Requests not classified as crawlers or scanners. |
| `vendors` | object | Map of vendor name to request count (e.g., `{"OpenAI": 1, "Google": 11}`). |
| `purposes` | object | Map of declared crawler purpose to count (e.g., `{"search": 6, "seo": 3}`). Present in some files. |
| `topPaths` | object | Map of URL path to request count, showing most-accessed paths. |

---

## 5. Honeypot Files (`honeypot_*.json`)

Top-level structure:

| Field | Type | Description |
|-------|------|-------------|
| `days` | integer | Number of days covered. |
| `count` | integer | Total number of honeypot hits. |
| `entries` | array | Array of honeypot hit records. |

Entry fields are identical to crawl log entries (Section 1) but contain only requests where `honeypot == true`. The three honeypot paths are:

- `/datasets/training-corpus-metadata.json` -- mimics a training data manifest
- `/research-notes/unpublished-draft-2026.html` -- mimics an unpublished manuscript
- `/data/model-evaluation-benchmark.csv` -- mimics a benchmark dataset

These paths are not linked from any page on the site. Any request to them indicates automated URL enumeration or targeted crawling.

---

## 6. Session Compliance Files (`sessions_*.json`)

Top-level structure: array of session objects.

### Session fields:

| Field | Type | Description |
|-------|------|-------------|
| `ip_hash` | string (16 hex chars) | SHA-256 hash of the client IP, truncated. Links to crawl log entries. |
| `first_seen` | string (ISO 8601) | Timestamp of the first request from this visitor. |
| `last_seen` | string (ISO 8601) | Timestamp of the last request from this visitor. |
| `sequence_count` | integer | Total number of requests in this session. |
| `compliance` | object | Opt-out compliance tracking (see below). |

### Compliance fields (`compliance`):

| Field | Type | Description |
|-------|------|-------------|
| `read_robots` | boolean | True if the visitor requested `/robots.txt` at any point during the session. |
| `read_tdmrep` | boolean | True if the visitor requested `/.well-known/tdmrep.json`. |
| `read_ai_policy` | boolean | True if the visitor requested `/ai-policy.html`. |
| `accessed_content_after_robots` | boolean | True if the visitor accessed content pages after reading robots.txt. |
| `accessed_content_without_robots` | boolean | True if the visitor accessed content pages without ever reading robots.txt. |

---

## Notes on Data Quality

1. **IP hashing**: All IP addresses are irreversibly hashed before logging. The same IP produces the same hash within a session, enabling session reconstruction without PII exposure.

2. **Crawler identification**: The 27-signature crawler identification system matches against known User-Agent tokens. Crawlers that spoof or omit identifying UA strings will appear as `crawler.detected == false`. This is a lower bound on actual AI crawler activity.

3. **Scanner classification**: Requests probing for common vulnerability paths (`.env`, `.git/config`, `phpinfo.php`, AWS credentials, etc.) are classified as scanners. This classification is path-based and may include legitimate security auditing tools.

4. **Timestamp precision**: All timestamps are in UTC. Cloudflare Workers provide millisecond-precision timestamps.

5. **Missing data**: The `rdns` field is null for the majority of entries because reverse DNS resolution was not performed for all requests (resource constraints on the free Cloudflare Workers tier). Where present, it provides additional verification of crawler identity.

6. **Duplicate snapshots**: Timestamped files (e.g., `crawl-log_20260615_034400.json`) are point-in-time KV store snapshots. The daily files (e.g., `crawl-log_2026-06-15.json`) are comprehensive daily aggregates and should be preferred for analysis.
