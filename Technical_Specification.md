# PolyPredict — Technical Specification

**Version:** 1.1 (diagram-link revision)  
**Date:** October 30, 2026  
**Team:** Vinesh Murikinati; Charlie Crocker  
**Traces to:** [Requirements_Document.md](Requirements_Document.md) **version 1.1 (2026-10-30)**. Requirement IDs and thresholds in this specification are defined by that version.

## 1. Version and Change Log

| Version | Date | Change | Author |
|---|---|---|---|
| 1.1 | 2026-10-30 | Linked actual D0/D1/D2 and peer review files; updated requirements baseline to v1.1 | Vinesh Murikinati and Charlie Crocker |
| 1.0 | 2026-10-30 | Consolidated D0 architecture, D1 data/API model, D2 failure cases, Week 8 implementation plan, and reviewed project constraints; linked all baseline IDs to planned tests | Vinesh Murikinati and Charlie Crocker |

**Baseline rule:** If requirements change, record a revision in both documents and preserve withdrawn IDs. Protocol details below are a **proposed implementation contract** where original design files do not specify exact JSON field names; reconcile with repository D1/D2 before coding.

## 2. Architecture Summary

PolyPredict is a local-first Python research application with a single local HTTP API and local database. The **Collector** subscribes to a free-tier Polymarket WebSocket and/or reads permitted REST market data, validates observations, and sends them to **Storage**. The **Shift Detector** compares observations from the same event across a defined time window. Significant shifts prompt the **AI Engine** to retrieve a bounded number of news/social records and run the local Qwen3.5 9B model. The engine returns structured company/sector hypotheses and source references. The **Signal Generator** combines these hypotheses with stock-market context, validates the output, and stores a research signal. If the user chooses to simulate an action, it creates a local **PaperTrade** record or uses a configured sandbox-only interface. The **Dashboard** queries the FastAPI HTTP API to display saved results; it never places a live trade.

The six D0 major components are Collector, Shift Detector, AI Engine, Signal Generator, Storage, and Dashboard. Validation, API routing, and paper-trade simulation are responsibilities at their boundaries rather than newly added top-level D0 components. Data flow follows **collect → validate → store → detect shift → add context → AI analyze → generate signal → optional paper trade → store → dashboard**. Every failure should produce an error-bearing SystemRun record, and invalid payloads must not enter the downstream pipeline. The application runs on a student's local computer to respect CON-01 and CON-03. The originally identified D0 interfaces I1–I12 remain the architectural baseline, with their precise names preserved in the existing diagram rather than invented here.

**Related design documents:** [D0 High-Level Design](Design_Diagrams/D0_High_Level_Design.md), [D1 Detailed Design](Design_Diagrams/D1_Detailed_Design.md), [D2 Detailed Design](Design_Diagrams/D2_Detailed_Design.md), and [Peer Review Notes](Design_Diagrams/Peer_Review_Notes.md). These files contain the existing architecture and detailed design; diagrams are linked instead of duplicated. Supporting source: [User_Stories.md](User_Stories.md) and the Week 8 tasks T1–T10.

**Responsibility assignment:** Vinesh owns Collector, Shift Detector, AI Engine and their integration (T2–T4, T6, T9). Charlie owns Storage, Signal Generator, paper-trade logic, Dashboard, and their testing (T1, T5, T7, T8, T10).

## 3. Component Interfaces and Data Formats

### 3.1 Boundary contracts

All timestamps are UTC ISO 8601 strings (`2026-10-30T14:00:00Z`). All IDs are nonempty strings, confidence/probability are numbers in [0,1], volume/price/quantity are nonnegative numeric values (quantity >0 for trades). JSON is UTF-8. Failures are **recorded**, not silently converted to successful signals.

| Boundary | Inputs and protocol | Outputs and format | Error behavior |
|---|---|---|---|
| Market provider → Collector | WebSocket events or permitted HTTPS/REST responses with source market ID, outcome probability, volume, time | Validated `MarketObservation` JSON | Reject malformed or stale records; backoff on provider rate-limit or disconnect; store run error |
| Collector → Storage | Internal Python call or local API with validated event/observation JSON | Stored `MarketEvent` and `MarketObservation` IDs | Validation failure returns 422; duplicate event/time is idempotent; DB failure records run error |
| Storage → Shift Detector | Internal read by event ID, time range; observation records | Ordered observation pair and flagged-shift JSON | Too little history ⇒ no shift, not a crash; malformed baseline ⇒ error |
| Shift Detector → AI Engine | Internal JSON `{event_id, baseline, current, delta, context_refs}` | Analysis job/input JSON | Unsupported/empty event skipped and logged |
| Context sources → AI Engine | HTTPS REST/news feeds or source-specific permitted retrieval; text, URL/ID, time | Up to 10 sanitized `NewsItem` records | Network timeout ⇒ partial context with warning; malicious text treated strictly as data |
| AI Engine → Signal Generator | Local model inference; schema-checked `AnalysisResult` JSON | Affected entity, explanation, source IDs, confidence if available | Timeout/invalid model JSON ⇒ analysis failed; no signal issued |
| Stock source → Signal Generator | HTTPS/REST price quote with ticker, price, time | Validated quote JSON | Missing/stale quote ⇒ no trade; optionally store research signal as unavailable for execution |
| Signal Generator → Storage/Paper simulation | Internal call / `POST /api/papertrades`, approved signal, quantity, simulated quote | `TradingSignal` or `PaperTrade` JSON | Invalid or live-only configuration ⇒ 422/403; never fall back to live endpoints |
| Storage/API → Dashboard | Local HTTP GET returning JSON lists | Read-only event, analysis, signal, trade and status views | 404 unknown ID; 503 store unavailable; show clear user error, not stale data as fresh |

### 3.2 Proposed JSON payload examples

The field names below define a consistent starting contract; verify exact D1/D2 schema names before implementation.

```json
{
  "event_id": "market-001",
  "observed_at": "2026-10-30T14:00:00Z",
  "probability": 0.63,
  "volume": 1200.0,
  "source": "polymarket"
}
```

```json
{
  "event_id": "market-001",
  "baseline_observation_id": "obs-100",
  "current_observation_id": "obs-101",
  "probability_delta": 0.12,
  "window_minutes": 60
}
```

```json
{
  "analysis_id": "analysis-001",
  "event_id": "market-001",
  "affected_entities": [{"ticker": "EXAMPLE", "sector": "example-sector"}],
  "explanation": "Hypothesis about possible sector exposure",
  "source_ids": ["news-001"],
  "model": "local-qwen"
}
```

```json
{
  "signal_id": "signal-001",
  "analysis_id": "analysis-001",
  "event_id": "market-001",
  "ticker": "EXAMPLE",
  "direction": "watch",
  "confidence": 0.72,
  "rationale": "Research rationale, not an execution instruction"
}
```

Allowed signal direction vocabulary must be finalized in the source schema. `EXAMPLE` is a test placeholder, not a stock recommendation. No raw external text may override model system instructions or authorize trades.

### 3.3 API endpoints

API base path is local `/api`. The three endpoints carried forward from D1 are `POST /api/shifts/analyze`, `GET /api/signals`, and `POST /api/papertrades`; other reads listed below are **proposed** routes for the dashboard. JSON request validation is implemented by FastAPI/Pydantic.

| Method / endpoint | Input | Success output | Failure behavior | References |
|---|---|---|---|---|
| `POST /api/shifts/analyze` | JSON event ID and optional UTC comparison window | 200 JSON `{event_id, shift_detected, analysis_id?}` | 400 malformed range; 404 event absent; 422 invalid JSON; 503 provider/model unavailable | REQ-02, REQ-03 |
| `GET /api/signals` | Optional `event_id`, `from_utc`, `to_utc`, `limit` (default 20, max 100) | 200 JSON array of stored signals | 400 invalid dates/range; 503 database unavailable | REQ-04, REQ-09 |
| `POST /api/papertrades` | JSON `{signal_id, quantity, simulated_price}` | 201 JSON saved PaperTrade ID and status | 403 live execution requested; 404 signal missing; 422 invalid quantity/price; 503 write failure | REQ-06, CON-02 |
| `GET /api/events` *(proposed)* | `limit` up to 100 | 200 JSON recent market events and observations | 400 invalid limit; 503 storage unavailable | REQ-05 |
| `GET /api/runs` *(proposed)* | Optional status and limit | 200 JSON runs and error categories | 400 invalid filter; 503 storage unavailable | REQ-07 |
| `GET /api/analyses` *(proposed)* | Event ID and limit | 200 JSON analysis records | 400 invalid limit; 503 storage unavailable | REQ-05 |

**Uniform error JSON:** `{ "error": {"code": "VALIDATION_ERROR", "message": "Invalid probability"}, "run_id": "run-001" }`. Never include credentials, raw request tokens, or stack traces in API responses. HTTP 429 upstream errors are logged and retried with bounded exponential backoff; 400/422 errors are not retried unchanged.

### 3.4 Database schema / persistence

The D1 entities are `MarketEvent`, `MarketObservation`, `NewsItem`, `AnalysisResult`, `TradingSignal`, `PaperTrade`, and `SystemRun`. SQLite is the local implementation; PostgreSQL is a future alternative. These are minimum logical columns, not a claim that SQL migration files already exist.

| Entity | Required logical fields and keys | Relationships / checks |
|---|---|---|
| MarketEvent | `event_id` PK, `source`, `title` | Stable provider identifier |
| MarketObservation | `observation_id` PK, `event_id` FK, `observed_at`, `probability`, `volume` | FK → MarketEvent; unique(event_id, observed_at); probability 0–1 |
| NewsItem | `news_id` PK, `event_id` FK, `source_ref`, `published_at`, `summary` | Source attribution required |
| AnalysisResult | `analysis_id` PK, `event_id` FK, `created_at`, `model`, `explanation`, `source_ids` | Validated output; event exists |
| TradingSignal | `signal_id` PK, `analysis_id` FK, `event_id` FK, `ticker`, `direction`, `confidence`, `rationale` | Confidence 0–1; analysis exists |
| PaperTrade | `papertrade_id` PK, `signal_id` FK, `quantity`, `simulated_price`, `created_at`, `status` | Quantity >0; never external live order ID |
| SystemRun | `run_id` PK, `started_at`, `finished_at`, `status`, `error_code` | Status includes success/failed; failure can be traced |

Database transactions are atomic for each stored signal/trade, foreign keys enabled, and error details sanitized. Historical stock quotes and complex backtesting tables may be added later by a versioned migration if REQ-10 is implemented.

## 4. Technology Stack

| Layer | Choice | Rationale linked to baseline |
|---|---|---|
| Programming/runtime | Python 3 | Common no-cost language for ingestion, model orchestration, and statistical processing (CON-01, REQ-01–03) |
| External market data | Polymarket free-tier WebSocket / permitted REST | Avoid paid feeds; accepts modest rate limits (CON-01, REQ-01) |
| HTTP API and validation | FastAPI + Pydantic | Typed JSON validation, consistent 422 errors, local API (REQ-04, NFR-02, CON-03) |
| Local AI | Qwen3.5 9B run locally | Limits external data exposure and API costs (REQ-03, CON-01, CON-03) |
| Persistence | SQLite in development | Single-machine persistence without a managed database bill (REQ-01, REQ-07, CON-01) |
| Future optional DB | PostgreSQL | Supports later multi-user deployment without changing logical entities (REQ-09; not release-critical) |
| UI | HTML, JavaScript, local dashboard | Lightweight read-only views (REQ-05, NFR-03) |
| External context / stock quotes | Permitted free-tier HTTPS APIs; provider choice pending | Source attribution and cost control (REQ-08, CON-01) |
| Paper simulation | Local simulated ledger; sandbox-only adapter optional | No production order route (REQ-06, CON-02) |
| Testing | pytest, FastAPI TestClient, mocked WebSocket/HTTP providers | Deterministic functional, resilience, and security verification (NFR-01–05, CON-02–03) |

## 5. Known Constraints and Consequences

| ID | Restated inherited constraint | Forces / rules out |
|---|---|---|
| CON-01 | Project development costs must remain under **$100 USD**. | Free software, free-tier provider access, local hosting; rules out paid feeds/managed infrastructure and low-latency high-frequency strategy |
| CON-02 | Trading is **paper-only**, with zero real capital. | Local simulation or approved sandbox endpoint only; rules out live brokerage routing and production credentials |
| CON-03 | Locally secure credentials and strict validation of REST/WebSocket data. | Environment-variable secrets, validation at ingestion/model boundaries, redacted logs; rules out plaintext secrets committed to Git or trusting external text as commands |

**Operational limits:** Provider quotas, source availability, local model hardware, and free-tier stock quotation limits may reduce throughput. Backoff prevents accidental API abuse. The AI hypothesis is explanatory, not a statement of causal or financial certainty. Source licenses and policies should be verified before production-like use. These constraints do not authorize changing the baseline scope without a logged revision.

## 6. Traceability Matrix

The column names and order below match the assignment exactly. All IDs in [Requirements_Document.md v1.0](Requirements_Document.md) appear exactly once. **Designed** means a design mapping exists, not that code or a test is complete. TC IDs are reserved for the Week 10 plan.

| Req ID | Description | Design Component (D0/D1/D2) | Status | Test Case |
|---|---|---|---|---|
| REQ-01 | Valid Polymarket observations persisted | D0 Collector → D1 MarketObservation/Storage | Designed | TC-001 |
| REQ-02 | Detect ≥0.10 probability shift in 60 min | D1 Shift Detector → D2 boundary cases | Designed | TC-002 |
| REQ-03 | Structured local AI analysis | D0 AI Engine → D1 AnalysisResult → D2 invalid output | Designed | TC-003 |
| REQ-04 | Complete, validated research signals | D0 Signal Generator → D1 TradingSignal | Designed | TC-004 |
| REQ-05 | Dashboard latest 20 items | D0 Dashboard → D1 Storage read interface | Designed | TC-005 |
| REQ-06 | Simulated trade records only | D2 paper-trading flow → D1 PaperTrade | Designed | TC-006 |
| REQ-07 | Failed runs and rejected input logged | D2 failure branches → D1 SystemRun | Designed | TC-007 |
| REQ-08 | Source-linked context records | D0 AI Engine context interface → D1 NewsItem | Designed | TC-008 |
| REQ-09 | Read-only signal filtering | D1 GET /api/signals → D0 Dashboard | Designed | TC-009 |
| REQ-10 | Simulated trade summary | D1 PaperTrade → D0 Signal Generator/Dashboard | Planned | TC-010 |
| NFR-01 | 95% local reads ≤2 seconds | D1 FastAPI read endpoints → D0 Dashboard | Planned | TC-011 |
| NFR-02 | Environment-only secrets and strict validation | D2 validation/error handling → D0 Collector/API | Designed | TC-012 |
| NFR-03 | 4/5 users finish task ≤90 sec | D0 Dashboard → D2 UX acceptance path | Planned | TC-013 |
| NFR-04 | ≥99/100 stored despite five disconnects | D2 retries/error paths → D0 Collector/Storage | Designed | TC-014 |
| NFR-05 | Audit all 25 simulated trade attempts | D2 paper-trade failure path → D1 SystemRun/PaperTrade | Planned | TC-015 |
| CON-01 | Budget < $100 | D0 local deployment and free-tier Collector → D1 SQLite | Designed | TC-016 |
| CON-02 | No live capital or live order route | D2 paper-trading guard → D1 PaperTrade | Designed | TC-017 |
| CON-03 | Local keys and validated source payloads | D2 validation and failure handling → D0 Collector/API | Designed | TC-018 |

**Orphan check:** 18 of 18 baseline IDs are mapped to named design elements; none are ORPHANED. Planned tests are not evidence of passing results. REQ-10 is assigned **Planned** because its backtesting/reporting specifics are deferred.

## 7. Advisor Review Record

**Review date:** October 28, 2026.  
**What was discussed:** The advisor reviewed the PolyPredict project design and gave positive feedback, saying that it looked good. The team did not report any requested technical changes.  
**Changes produced by the review:** None required or recorded. This document's consolidation, measurable target selection, and traceability matrix are Assignment 9 documentation work and should **not** be misrepresented as changes requested or approved by the advisor.

The review covered the project design; it has **not been independently established** that the advisor reviewed these finished version 1.1 documents. The team should share the completed documents for any required follow-up review.

**References:** [Requirements_Document.md v1.1](Requirements_Document.md); [User_Stories.md](User_Stories.md); [D0 High-Level Design](Design_Diagrams/D0_High_Level_Design.md), [D1 Detailed Design](Design_Diagrams/D1_Detailed_Design.md), [D2 Detailed Design](Design_Diagrams/D2_Detailed_Design.md), and [Peer Review Notes](Design_Diagrams/Peer_Review_Notes.md); `Constraints_Essay.md`; D0, D1, D2 design source files; Week 8 implementation plan. Links to the existing individual design files are used instead of duplicating drawings.
