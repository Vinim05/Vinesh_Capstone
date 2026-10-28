# Table 1 — Timeline

**Basis:** Proposed Spring 2027 January 11–April 23. Each task is a 10-workday (two-week) slot with **eight working days planned plus two working days reserved (20%)** for task-level schedule contingency. The completion date includes its task buffer. Parallel tasks use mock contracts until upstream components are available; integration still respects dependencies.

| ID | Task statement (matches Tasklist.md) | Owner | Start | Completion incl. 20% buffer | Dependencies |
|---|---|---|---|---|---|
| T1 | Develop the Data Storage database schema, relationships, and API methods for saving market observations, analyses, trading signals, and paper trades. | Charlie | Jan 11 | Jan 22 | D1 data model |
| T2 | Develop the Prediction Market Data Collector to retrieve, validate, and store Polymarket observations through the API. | Vinesh | Jan 11 | Jan 22 | D2 collector contract; mock T1 persistence until integration |
| T3 | Develop the Sentiment Shift Detector to compare market probabilities and trading volume and identify significant changes. | Vinesh | Jan 25 | Feb 5 | T1, T2 |
| T4 | Develop the AI Analysis Engine using the local Qwen3.5 9B model to identify potentially affected companies and industries. | Vinesh | Jan 25 | Feb 5 | D1 AI contract; fixture shifts; T3 required for real input |
| T5 | Develop the Trading Signal Generator to combine AI analysis with stock market data and generate trading recommendations. | Charlie | Feb 8 | Feb 19 | T1, T3, T4 |
| T6 | Develop the AI Analysis Engine's news and social context integration to support analysis of detected market shifts. | Vinesh | Feb 8 | Feb 19 | T3, T4 |
| T7 | Develop the Trading Signal Generator's paper trading and historical backtesting functionality to evaluate generated signals. | Charlie | Feb 22 | Mar 5 | T1, T5 |
| T8 | Develop the User Dashboard to display market observations, sentiment shifts, AI analysis, trading signals, and simulated results. | Charlie | Feb 22 | Mar 5 | T1, T5; mock T7 results until available |
| T9 | Test and refine the Prediction Market Data Collector, Sentiment Shift Detector, and AI Analysis Engine integration with the complete processing pipeline. | Vinesh | Mar 8 | Mar 19 | T2, T3, T4, T5, T6, T7, T8 |
| T10 | Test and validate the User Dashboard, Data Storage, and Trading Signal Generator using the project acceptance criteria. | Charlie | Mar 22 | Apr 2 | T1, T5, T7, T8, T9 |

**Milestone completion dates (separate rows, as required):**

| Milestone | Outcome | Planned completion |
|---|---|---|
| F1–F14 | Fall deliverables | Aug 28, Sep 4, Sep 11, Sep 18, Sep 25, Oct 2, Oct 9, Oct 16, Oct 23, Oct 30, Nov 6, Nov 13, Nov 20, Dec 11, 2026 (respectively; dates estimated) |
| S1 | Collection and persistence verified | Jan 22, 2027 |
| S2 | Detector and baseline AI verified | Feb 5, 2027 |
| S3 | Context and signal verified | Feb 19, 2027 |
| S4 | Paper/backtest and UI verified | Mar 5, 2027 |
| S5 | End-to-end integration verified | Mar 19, 2027 |
| S6 | Acceptance tests verified | Apr 2, 2027 |
| S7 | Correction buffer / demo freeze | Apr 16, 2027 |
| S8 | Final handoff | Apr 23, 2027 |

**Protected milestone buffers:** Jan 21–22 (S1), Feb 4–5 (S2), Feb 18–19 (S3), Mar 4–5 (S4), Mar 18–19 (S5), Apr 1–2 (S6) are protected task-level contingency days and cannot be assigned new features. Additionally, **Apr 5–9** is a protected five-workday block for regression fixes, API/provider issues, and hardware failures; **Apr 12–16** is a protected five-workday block for final demonstration stabilization and advisor review before S8. Apr 19–23 is reserved for the final demonstration, handoff, and deadline adjustments, not new features. These late blocks are distinct from each task's 20% buffer.

**Critical path / high-impact dependencies:** T1 + T2 → T3 → T5 → T7 → T9 → T10 → S6 → protected buffer → S8. T4 and T6 are parallel but gate T5/T9. Any failure of stock data or paper-trading API (R2), AI hardware (R3), or member availability (R6) may delay a path-gating task; their buffers and fallback actions are reserved above.

**Calendar constraints:** Winter break Dec 12–Jan 10: no planned tasks. Spring exams, co-ops, and exact university break dates remain unconfirmed and must be reconciled before adopting this as a team commitment.
