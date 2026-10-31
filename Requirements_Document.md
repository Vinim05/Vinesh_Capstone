# PolyPredict — Requirements Document

**Version:** 1.1 (diagram-link revision)  
**Date:** October 30, 2026  
**Team:** Vinesh Murikinati; Charlie Crocker  
**Project:** Prediction Market Sentiment Trading System (PolyPredict)  
**Companion:** [Technical_Specification.md](Technical_Specification.md), version 1.1

## 1. Version and Change Log

| Version | Date | Change | Author |
|---|---|---|---|
| 1.1 | 2026-10-30 | Corrected direct design document links and aligned companion version | Vinesh Murikinati and Charlie Crocker |
| 1.0 | 2026-10-30 | Consolidated Week 4 stories, D0–D2 design, Week 8 plan, and Week 3 constraints; established stable verification IDs and proposed numerical acceptance targets | Vinesh Murikinati and Charlie Crocker |

**ID policy:** REQ, NFR, and CON identifiers are permanent. A removed item remains recorded as **Withdrawn** in this table and its change is logged; identifiers are not reused. No items are withdrawn in v1.0. The measurable thresholds below are **design targets**, not claims of measured implementation performance.

## 2. Purpose and Scope

PolyPredict is a locally run academic research tool that observes Polymarket prediction-market probability and volume changes, identifies meaningful shifts, collects contextual news or social information, uses a local AI model to suggest potentially affected industries or listed companies, and generates explainable stock-related signals. It stores inputs and outputs for review and supports **simulated paper trades only**. The project helps researchers examine possible relationships between prediction-market sentiment and equity markets; it does not claim that signals predict prices accurately or constitute investment advice.

**In scope:** ingestion and validation, historical observations, shift detection, contextual AI analysis, stock-market context, signal records, paper-trade records, error/audit records, and a read-only research dashboard. The six major D0 components are Collector, Shift Detector, AI Engine, Signal Generator, Storage, and Dashboard. **Out of scope:** live-money trading, order routing to production brokerage endpoints, high-frequency execution, paid market feeds, and guaranteed investment returns.

The reference baseline is Week 4 user stories and use case UC-01, D0 architecture/interfaces I1–I12, D1 storage and component contracts, D2 edge cases/traceability, Week 8 task and milestone plan, and the Week 3 constraints essay. See [User_Stories.md](User_Stories.md) and [D0 High-Level Design](Design_Diagrams/D0_High_Level_Design.md), [D1 Detailed Design](Design_Diagrams/D1_Detailed_Design.md), [D2 Detailed Design](Design_Diagrams/D2_Detailed_Design.md), and [Peer Review Notes](Design_Diagrams/Peer_Review_Notes.md). These existing design documents are linked rather than reproduced.

## 3. Stakeholders and User Goals

| Stakeholder / role | Goal | Origin |
|---|---|---|
| Market researcher (primary) | Investigate meaningful prediction-market probability changes, AI context, and historical evidence without executing real trades | Week 4, US-01, UC-01 |
| System administrator (secondary) | Configure data sources and credentials, observe failures, and recover from bad inputs or unavailable providers | Week 4, US-02 |
| Compliance reviewer (hidden) | Confirm that paper-only boundaries, source attribution, and the audit trail are maintained | Week 4, US-03 |
| Project team | Build and test an affordable, local proof of concept during Spring 2027 | Week 8, T1–T10 |

**User-goal interpretation:** US-01 maps to research and signal visibility; US-02 maps to configuration and system monitoring; US-03 maps to paper-only and auditability. Check exact wording against the repository's authoritative `User_Stories.md` before final commit.

## 4. Glossary

| Term | Definition |
|---|---|
| Prediction market | Market whose quoted prices/probabilities express participants' assessment of an event outcome |
| Market event | A tracked question or contract identified by its external source identifier |
| Market observation | A timestamped probability and volume reading for a market event |
| Sentiment shift | A configured change in probability and/or volume over a comparison window |
| News item | Timestamped text, link, and source metadata used as context, not an instruction to the system |
| AI analysis | Local model output linking a shift to possible companies, sectors, and an explanation |
| Trading signal | A research recommendation with a company symbol, direction, rationale, confidence, and provenance |
| Paper trade | Simulated trade recorded without an order sent to a live execution system |
| System run | Tracked execution of the ingestion-to-signal pipeline, including failures |
| Dashboard | Browser interface for reviewing saved observations, signals, and simulated outcomes |
| Market researcher | Primary user who reviews relationships and signal evidence |
| System administrator | User responsible for local configuration, monitoring, and troubleshooting |
| Compliance reviewer | Stakeholder checking paper-only and data-handling controls |
| MoSCoW | Prioritization scheme: Must, Should, Could, and Won't for this release |
| UC-01 | Week 4 core end-to-end research workflow |
| API | Application programming interface |
| NFR | Non-functional requirement with a measurable quality target |

## 5. Functional Requirements

Every row is an acceptance criterion. Source tags refer to the Week 4 story/function and related D0–D2 or Week 8 design work; acceptance tests are planned for Week 10. **Must = four of ten functional requirements**, so the MVP is intentionally limited.

| ID | Requirement statement | MoSCoW | Source story | Verification method |
|---|---|---|---|---|
| REQ-01 | **The system shall** ingest a valid probability (0–1), volume (≥0), source ID, and UTC timestamp for at least one selected Polymarket event and persist one observation per received unique event/timestamp pair. | Must | US-01; D0 Collector; T2 | TC-001: valid and duplicate sample integration tests |
| REQ-02 | **The system shall** compare the newest observation to a stored baseline and flag a shift when absolute probability change is ≥0.10 within a 60-minute window; each flagged shift shall reference both observations. | Must | US-01; D1 Shift Detector; T3 | TC-002: boundary-value tests (0.099/0.100) |
| REQ-03 | **The system shall** generate and persist one structured analysis with at least one candidate company/sector, explanatory text, and source references for each valid shift it successfully processes. | Must | US-01; D1 AI Engine; T4 | TC-003: schema and mock-model output tests |
| REQ-04 | **The system shall** create a signal record containing event ID, ticker, direction, numeric confidence (0–1), rationale, and analysis ID; it shall emit no actionable signal if required fields are missing. | Must | US-01; D1 Signal Generator; T5 | TC-004: required-field and negative tests |
| REQ-05 | **The system shall** show the 20 most recent stored events, shifts, analyses, and signals in a dashboard, with their timestamps and underlying rationale. | Should | US-01; D0 Dashboard; T8 | TC-005: UI view and ordering checks |
| REQ-06 | **The system shall** support creation of a simulated trade for a valid signal and persist its quantity, simulated price, time, and outcome without sending any order to a live brokerage endpoint. | Should | US-03; D2 paper-trade flow; T7 | TC-006: sandbox mock and network-call assertions |
| REQ-07 | **The system shall** save a run status and error category for every rejected external payload or failed processing step, without inserting incomplete analysis or signal records. | Should | US-02; D2 error paths; T9 | TC-007: fault injection and database inspection |
| REQ-08 | **The system shall** retrieve up to 10 timestamped news/social context records for a flagged event and attribute every retained record to a source URL or provider ID. | Should | US-01; D0 news context; T6 | TC-008: mock context retrieval and provenance checks |
| REQ-09 | **The system shall** support read-only filtering of stored signals by event ID and UTC time range, returning zero or more matches without modifying stored records. | Could | US-01; D1 Storage/Dashboard; T8 | TC-009: API filtering and integrity tests |
| REQ-10 | **The system shall** produce a repeatable paper-trade summary for a selected date range showing trade count and total simulated profit/loss calculated from saved prices. | Could | US-01; Week 8 T7 | TC-010: deterministic backtest fixture |

## 6. Non-Functional Requirements

**Targets are proposed acceptance thresholds** for the teaching project, not previously validated field measurements. Test on one documented local development computer using 100 seeded observations unless a row states otherwise.

| ID | Requirement statement | MoSCoW | Source story | Verification method |
|---|---|---|---|---|
| NFR-01 | **The system shall** respond to 95% of local read-only requests for 100 stored events within 2.0 seconds, measured over 100 sequential requests on the documented test machine. | Should | US-01; D1 API; T8 | TC-011: timed API benchmark |
| NFR-02 | **The system shall** load API credentials only from local environment configuration, never include credentials in HTTP responses or persisted run logs, and reject 100% of 20 crafted malformed input payloads in the security test set. | Must | US-02/US-03; D2 validation; Week 3 security | TC-012: secret-scan, malformed-input tests |
| NFR-03 | **The system shall** allow at least 4 of 5 first-time test participants to locate a signal and view its rationale within 90 seconds from opening the dashboard. | Could | US-01; D0 Dashboard; T8 | TC-013: moderated usability exercise |
| NFR-04 | **The system shall** persist at least 99 of 100 valid seeded observations under a test that includes five temporary upstream disconnects; a failed run must record an error and must not corrupt previously stored observations. | Should | US-02; D2 retries; T9 | TC-014: resilience and recovery test |
| NFR-05 | **The system shall** record an audit entry with run ID, UTC time, and outcome for 100% of 25 tested simulated-trade attempts, including failures. | Should | US-03; D2 paper-trade flow | TC-015: audit coverage test |

## 7. MoSCoW Summary

**Must (MVP):** REQ-01 ingestion; REQ-02 shift detection; REQ-03 analysis; REQ-04 signal records; NFR-02 credential and payload controls.

**Should:** REQ-05 dashboard; REQ-06 paper-trade creation; REQ-07 error records; REQ-08 source context; NFR-01 response-time target; NFR-04 recovery; NFR-05 audit coverage.

**Could:** REQ-09 signal filters; REQ-10 simulated-profit summary; NFR-03 usability study.

**Won't this release:** live trading or live order placement; high-frequency market execution; paid data feeds; automated brokerage account funding; guarantees about trade profitability. These are deliberate exclusions rather than deferred, untracked requirements.

## 8. Inherited Constraints

| ID | Inherited constraint (Week 3) | Consequence for the solution | Verification |
|---|---|---|---|
| CON-01 | Development spending must remain **under $100 USD**. | Use free-tier Polymarket access and freeware Python tools; no subscription-only services; accept lower throughput and frequency. | TC-016: budget ledger and dependency review |
| CON-02 | All trading must be **simulated only**, with no real capital or production order routing. | Only local simulation or explicitly sandboxed APIs; prohibit any live execution credentials/endpoints. | TC-017: configuration and outbound-call audit |
| CON-03 | API credentials must remain local, and untrusted WebSocket/REST content must be validated. | Environment-based secrets, strict schema validation, and rejection of malicious/malformed source data. | TC-018: threat-oriented configuration and injection tests |

The constraints are inherited from `Constraints_Essay.md`; they are not additional redesign proposals. Regulatory compliance and third-party terms require checking current provider rules before deployment; this document does not certify compliance.

## 9. Assumptions and Open Questions

| Item | Assumption or question | Owner | Review by |
|---|---|---|---|
| A-01 | Local FastAPI/SQLite deployment remains the development baseline; Postgres is a possible later production substitute, not required for this release. | Charlie | 2026-11-06 |
| A-02 | Qwen3.5 9B is run locally; models may fail or return invalid JSON and must be handled as failed analysis. | Vinesh | 2026-11-06 |
| A-03 | Thresholds in REQ-02 and NFR-01–05 are proposed and can be revised by logged version changes before Week 10 acceptance tests. | Vinesh and Charlie | 2026-11-06 |
| Q-01 | Confirm the exact stock-price source and any service-specific response fields permitted under free-tier terms. | Charlie | 2026-11-06 |
| Q-02 | Confirm whether paper trades are entirely local or sent only to an Alpaca sandbox; never enable live routing. | Charlie | 2026-11-06 |
| Q-03 | Cross-check story wording and diagram filenames against authoritative repository source before committing hyperlinks. | Vinesh | 2026-11-06 |

**Related source:** [Technical specification v1.0](Technical_Specification.md). Verification IDs TC-001–TC-018 are reservations for the Week 10 test plan; no tests are represented as already passed.
