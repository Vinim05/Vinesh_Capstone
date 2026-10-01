# User Stories

## Stakeholder Map

- **Primary:** Market researcher/trader — uses the system to identify prediction market changes that may affect stocks or industries.
- **Secondary:** System administrator/developer — maintains the system and ensures market data is collected correctly.
- **Hidden:** Compliance/risk reviewer — reviews the system's analysis and needs to trace signals back to the data that produced them.

## User Stories

### US-01 — Primary

As a **market researcher**,  
I want to **identify significant shifts in prediction market sentiment**,  
so that I can **investigate events that may affect publicly traded companies or industries**.

### US-02 — Secondary

As a **system administrator**,  
I want to **record prediction market and stock market data with timestamps and source information**,  
so that I can **verify the data used in an analysis and reproduce historical tests**.

### US-03 — Hidden

As a **compliance/risk reviewer**,  
I want to **trace each generated market signal back to the prediction market data and AI analysis that produced it**,  
so that I can **review the evidence behind a signal before it is used for financial analysis**.

# Use Cases

## UC-01 — Analyze a Market Shift

**Expands:** US-01  
**Primary Actor:** Market researcher  
**Secondary Actors:** Polymarket, stock market data source, AI service

### Preconditions

1. Polymarket and stock market data are available.
2. Previous market data exists for comparison.
3. The AI service is available.

### Main Success Flow

1. **Actor:** The researcher starts the analysis.
2. **System:** The system compares current Polymarket data with previous data.
3. **Actor:** The researcher requests analysis of a detected shift.
4. **System:** The system uses AI to analyze the shift and find potentially affected companies.
5. **Actor:** The researcher selects the result for backtesting.
6. **System:** The system compares the result with stock market data and saves the results.

### Alternate Flow — No Significant Shift

1. No significant market shift is detected.
2. The system saves the data and continues monitoring.

### Exception Flow — AI Service Unavailable

1. A significant shift is detected, but the AI service fails.
2. The system records the failure and saves the market data.

### Postcondition

The market data and any completed analysis or backtesting results are saved.

# Acceptance Criteria

### AC-01.1 — Main Success Flow

**Given** a significant market shift is detected,  
**When** the researcher requests an analysis,  
**Then** the system identifies at least one potentially affected company or industry.

### AC-01.2 — Exception Flow

**Given** a significant market shift is detected,  
**When** the AI service fails,  
**Then** the system saves the market data and records the analysis as failed.
