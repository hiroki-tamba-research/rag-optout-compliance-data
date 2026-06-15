# RAG-Level Opt-Out Compliance Failure: Empirical Evidence from Four Major AI Vendors

## Description

This dataset provides empirical evidence that all four major AI vendors (Google, OpenAI, Anthropic, and xAI) fail to respect content opt-out signals at the retrieval-augmented generation (RAG) level, even when those same vendors correctly honour opt-out at the crawler level.

The test site (tlimresearch.org) was instrumented with three layers of machine-readable opt-out:

1. `noai` and `noimageai` HTML meta tags (following the proposed C2PA/IPTC convention)
2. TDM (Text and Data Mining) reservation via `/.well-known/tdmrep.json` (per W3C TDM Reservation Protocol)
3. `robots.txt` directives disallowing known AI training crawlers

On 2026-06-15, all four vendors' RAG-enabled interfaces (Google AI Mode, ChatGPT with web search, Grok/SuperGrok, and Claude with URL fetching) were queried about content hosted on tlimresearch.org. All four retrieved, processed, and presented content from the site in their responses, despite the opt-out signals. Separately, crawler-level bots from OpenAI and Anthropic were observed checking robots.txt and respecting its directives -- demonstrating that crawler-level opt-out functions correctly but no equivalent mechanism exists at the RAG/inference layer.

The dataset also includes canary token baselines establishing that none of the site content appeared in any vendor's pre-training data, confirming that all vendor knowledge of the site was acquired exclusively through real-time RAG retrieval.

## Temporal Coverage

- Crawl log collection period: 2026-06-11 through 2026-06-15
- RAG opt-out violation test: 2026-06-15
- Canary token baselines: 2026-06-15

## Methods

### Site Instrumentation

The test site (tlimresearch.org) was deployed on Cloudflare Workers with:

- **HTML meta tags**: `<meta name="robots" content="noai, noimageai">` on all pages
- **TDM reservation**: `/.well-known/tdmrep.json` declaring TDM rights reservation under the W3C TDM Reservation Protocol
- **robots.txt**: Disallow directives for known AI training crawlers (GPTBot, CCBot, Google-Extended, anthropic-ai, etc.)
- **Crawl tracker**: A Cloudflare Worker logging all HTTP requests with IP hashing (SHA-256, truncated), user-agent parsing, ASN lookup, and crawler identification for 27 known AI crawler signatures
- **Honeypot paths**: Three invisible paths (`/datasets/training-corpus-metadata.json`, `/research-notes/unpublished-draft-2026.html`, `/data/model-evaluation-benchmark.csv`) not linked from any page, designed to detect automated deep crawling

### Canary Token Design

Seven fabricated academic-sounding terms were embedded in site content and queried against each vendor's API in pre-training-only mode (no web search, no RAG, no connectors). These serve as negative controls: if a vendor reports familiarity with a canary token, it indicates the site content has entered pre-training data.

Canary tokens used: Bramblecroft convention, Silvermark-Tamba recursion, Hazelbrook-Tamba threshold, Pennworth case-sensitivity index, Blackthorn-Renfield calibration, Tamba-Vestridge index (Cormorant schema), Whittaker-Tamba coefficient (Protocol Ashgrove-7).

### RAG Violation Testing

Each vendor was queried via its RAG-enabled consumer interface about content from tlimresearch.org:

- **Google**: AI Mode in Google Search (standard web query)
- **OpenAI**: ChatGPT temporary chat with web search enabled, history/memory/connectors disabled
- **Anthropic**: Claude incognito chat (Opus 4.6 Max), URL provided in query
- **xAI**: Grok via SuperGrok with web search enabled

### Privacy Protections Applied to Data

- All IP addresses are SHA-256 hashed and truncated to 16 hex characters before logging
- No raw IP addresses appear in any data file
- User-agent strings are logged verbatim (these are self-declared by clients and do not constitute PII)
- Screenshot files included in the evidence directory may contain the researcher's browser UI elements but no third-party PII

## File Descriptions

### Canary Token Baselines

| File | Description |
|------|-------------|
| `canary_baseline_anthropic_2026-06-15.json` | Anthropic API (claude-sonnet-4-6) canary token baseline. 7/7 negative. Direct API call, no system prompt, no web retrieval. |
| `canary_baseline_grok_2026-06-15.json` | xAI API (grok-3) canary token baseline. 7/7 negative. Direct API call, no system prompt, no web retrieval. |
| `canary_daily_anthropic_2026-06-15.json` | Anthropic daily canary monitor run (automated cron). 7/7 negative. Includes response hashes for longitudinal comparison. |

Google and OpenAI canary baselines were collected via browser interface (screenshots only; no programmatic API baseline files for these vendors). Screenshot evidence is in the `evidence/` directory.

### Crawl Logs

| File | Description |
|------|-------------|
| `crawl-log_2026-06-11.json` | Full crawl log for 2026-06-11 (site launch day). |
| `crawl-log_2026-06-12.json` | Full crawl log for 2026-06-12. |
| `crawl-log_2026-06-13.json` | Full crawl log for 2026-06-13. |
| `crawl-log_2026-06-14.json` | Full crawl log for 2026-06-14. |
| `crawl-log_2026-06-15.json` | Full crawl log for 2026-06-15 (RAG violation test day). |

Each crawl log is a JSON object containing an array of HTTP request entries with crawler identification, ASN attribution, honeypot flags, and compliance tracking fields. See DATA_DICTIONARY.md for field-level documentation.

### Aggregate Statistics

| File | Description |
|------|-------------|
| `crawl-stats_full_20260614.json` | 7-day aggregate statistics (2026-06-08 through 2026-06-14). Vendor breakdown, path frequency, scanner vs. legitimate traffic counts. |
| `stats_20260615_213746.json` | Single-day statistics for 2026-06-15 (1,438 total requests; 19 identified crawlers; vendors: OpenAI 1, Google 11, Moz 3, Microsoft 3, Anthropic 1). |

### Honeypot Data

| File | Description |
|------|-------------|
| `honeypot_20260613_142643.json` | Honeypot hits for 2026-06-13. |
| `honeypot_20260614_135730.json` | Honeypot hits for 2026-06-14. |
| `honeypot_20260614_210059.json` | Honeypot hits for 2026-06-14 (second pull). |
| `honeypot_20260615_*.json` | Honeypot hits for 2026-06-15 (multiple pulls throughout the day). |

Honeypot files contain only requests to paths that are not linked from any page on the site, indicating automated deep crawling or URL enumeration.

### Session Compliance Data

| File | Description |
|------|-------------|
| `sessions_2026-06-11.json` | Per-visitor session compliance data for 2026-06-11. |
| `sessions_2026-06-12.json` | Per-visitor session compliance data for 2026-06-12. |
| `sessions_2026-06-13.json` | Per-visitor session compliance data for 2026-06-13. |
| `sessions_2026-06-14.json` | Per-visitor session compliance data for 2026-06-14. |

Session files track whether each unique visitor (by hashed IP) checked robots.txt, tdmrep.json, or ai-policy.html before accessing content.

### Timestamped Snapshots

Files with timestamps in their names (e.g., `crawl-log_20260615_034400.json`) are point-in-time snapshots taken at the indicated time. These capture the state of the Cloudflare Workers KV store at that moment. The daily files (e.g., `crawl-log_2026-06-15.json`) are comprehensive daily aggregates.

## Key Findings

1. **4/4 vendors violated RAG-level opt-out**: Google, OpenAI, Anthropic, and xAI all retrieved and used content from a site with noai meta tags and TDM reservation via their RAG-enabled interfaces.
2. **Crawler-level opt-out works**: OpenAI (OAI-SearchBot) and Anthropic crawlers were observed checking robots.txt. No content pages were accessed by identified vendor crawlers that were disallowed.
3. **No RAG-level opt-out mechanism exists**: None of the four vendors checked noai meta tags, TDM reservation, or any other machine-readable signal before using site content in RAG responses.
4. **Pre-training baseline clean**: All 7 canary tokens returned negative across all 4 vendors, confirming zero pre-training contamination.
5. **Fabrication under uncertainty**: When OpenAI (ChatGPT) encountered an unfamiliar canary token (Silvermark-Tamba recursion), it fabricated numerical specifications ("3-5 layers, Hard stop: 7 layers") for the non-existent concept. Google (Gemini AI Mode) reported "Four-Layer" when the correct name is "Tri-Layer", fabricating from a noai-protected source.

## License

This dataset is published under CC0 1.0 Universal (Public Domain Dedication) as required by Dryad's data sharing policy.

Note: While the raw data is CC0, the associated analysis, publications, and software tools are separately licensed under CC-BY-NC-ND-4.0. TDM (Text and Data Mining) rights for the source website (tlimresearch.org) remain reserved per W3C TDM Reservation Protocol.

## Related Publications

- Tamba, H. (in preparation). RAG-level opt-out compliance failure across major AI vendors. Correspondence submitted to *Nature Machine Intelligence*.
- Tamba, H. (2026). LLM-as-judge non-determinism under controlled conditions. Zenodo. https://doi.org/10.5281/zenodo.20674090
- Tamba, H. (2026). Observer-aware red-teaming protocol for AI system evaluation. Zenodo. https://doi.org/10.5281/zenodo.20612989

## Contact

Hiroki Tamba
Tamba Research Academy
Email: contact@tamba-research.academy
ORCID: https://orcid.org/0009-0004-7635-0741

## Citation

If you use this dataset, please cite:

> Tamba, H. (2026). RAG-Level Opt-Out Compliance Failure: Empirical Evidence from Four Major AI Vendors [Dataset]. Dryad. https://doi.org/[DOI-pending]
