# D0 High-Level Design

**Basic Input and Output:** The system takes prediction market data from Polymarket, relevant news and social media information, and stock market data as input. The system outputs AI-generated company or industry impact analysis, trading signals, and paper trading results.

## Block Diagram (D0 Diagram)

```mermaid
flowchart TD

    %% Diagram Title and Goal
    subgraph HEADER["Prediction Market Sentiment Trading System — Block Diagram"]
        GOAL["Goal: Show the major system components, external systems, and the interfaces connecting them."]
    end

    %% External Systems
    PM[Polymarket API]
    NEWS[News / Social Media Sources]
    STOCK[Stock Market Data API]
    PAPER[Paper Trading API]

    %% Internal Components
    COLLECT[Prediction Market Data Collector]
    SHIFT[Sentiment Shift Detector]
    AI[AI Analysis Engine<br/>Local Qwen3.5 9B]
    SIGNAL[Trading Signal Generator]
    STORE[Data Storage]
    DASH[User Dashboard]

    %% Interfaces
    PM -->|I1: Prediction market data| COLLECT

    COLLECT -->|I2: Validated market records| SHIFT

    SHIFT -->|I3: Detected sentiment shift| AI

    NEWS -->|I4: Relevant news / sentiment data| AI

    AI -->|I5: Company / industry impact analysis| SIGNAL

    STOCK -->|I6: Stock price and volatility data| SIGNAL

    SIGNAL -->|I7: Paper trade request| PAPER

    PAPER -->|I8: Paper trade result| STORE

    COLLECT -->|I9: Market history| STORE

    AI -->|I10: AI analysis results| STORE

    SIGNAL -->|I11: Generated trading signals| STORE

    STORE -->|I12: Stored analysis and performance data| DASH

    %% Legend
    subgraph LEGEND["Legend"]
        L1[Internal Component]
        L2[External System / API]
        L3["Arrow = Data transferred between components"]
        L4["I# = Interface ID"]
    end

    %% Styles
    classDef internal fill:#D6E8FF,stroke:#2563EB,stroke-width:2px,color:#111827;
    classDef external fill:#FFE5CC,stroke:#EA580C,stroke-width:2px,color:#111827;
    classDef interface fill:#F3F4F6,stroke:#6B7280,stroke-width:1px,color:#111827;
    classDef header fill:#FFF4CC,stroke:#CA8A04,stroke-width:1px,color:#111827;

    class COLLECT,SHIFT,AI,SIGNAL,STORE,DASH,L1 internal;
    class PM,NEWS,STOCK,PAPER,L2 external;
    class L3,L4 interface;
    class GOAL header;
```


## Component Responsibility Table

| Component | Responsibility | Interfaces In | Interfaces Out | Primary Owner |
|---|---|---|---|---|
| Prediction Market Data Collector | Collects and validates prediction market data from Polymarket. | I1 | I2, I9 | Vinesh |
| Sentiment Shift Detector | Detects significant changes in prediction market sentiment from validated market records. | I2 | I3 | Vinesh |
| AI Analysis Engine | Uses the local Qwen3.5 9B model to identify companies or industries potentially affected by a detected sentiment shift. | I3, I4 | I5, I10 | Vinesh |
| Trading Signal Generator | Generates a paper trading signal using the AI impact analysis and current stock market data. | I5, I6 | I7, I11 | Charlie |
| Data Storage | Maintains persistent records produced by the system for later retrieval. | I8, I9, I10, I11 | I12 | Charlie |
| User Dashboard | Displays stored analysis and paper trading performance data to the user. | I12 | None | Charlie |

## Data-Flow Diagram

```mermaid
flowchart TD

    %% Diagram Title and Goal
    subgraph HEADER["Prediction Market Sentiment Trading System — Data-Flow Diagram"]
        GOAL["Goal: Show how data moves through the system and changes from raw market data into trading signals and paper trading results."]
    end

    %% External Systems
    PM[Polymarket API]
    NEWS[News / Social Media Sources]
    STOCK[Stock Market Data API]
    PAPER[Paper Trading API]

    %% Main Processing Flow
    subgraph FLOW1["Flow 1: Market Analysis and Paper Trading"]
        RAW[Raw Prediction Market Data]
        VALID[Validated Market Record]
        SHIFT[Detected Sentiment Shift]
        CONTEXT[AI Analysis Context]
        IMPACT[Company / Industry Impact Analysis]
        SIGNAL[Generated Trading Signal]
        TRADE[Paper Trade Result]
    end

    %% Storage Flow
    subgraph FLOW2["Flow 2: Storage and Reporting"]
        STORED[Stored Historical / Performance Record]
        DASH[Dashboard Display]
    end

    %% Main Data Flow
    PM -->|Raw probability, volume, timestamp| RAW

    RAW -->|Validated probability, volume, timestamp| VALID

    VALID -->|Probability and volume changes| SHIFT

    SHIFT -->|Topic, probability change, volume change, timestamp| CONTEXT

    NEWS -->|Relevant headlines and sentiment information| CONTEXT

    CONTEXT -->|Structured AI analysis input| IMPACT

    IMPACT -->|Affected companies, industries, expected impact| SIGNAL

    STOCK -->|Stock price and volatility data| SIGNAL

    SIGNAL -->|Ticker, action, confidence, reasoning| PAPER

    PAPER -->|Simulated order status, price, and position| TRADE

    %% Storage and Reporting Flow
    VALID -->|Validated market record| STORED

    IMPACT -->|AI analysis record| STORED

    SIGNAL -->|Generated signal record| STORED

    TRADE -->|Paper trade performance record| STORED

    STORED -->|Stored market, analysis, signal, and performance data| DASH

    %% Legend
    subgraph LEGEND["Legend"]
        L1[External Data Source / API]
        L2[Data / Processing Stage]
        L3["Arrow = Data transferred and its form"]
        L4["Flow 1 = Analysis and paper trading"]
        L5["Flow 2 = Storage and reporting"]
    end

    %% Styles
    classDef external fill:#FFE5CC,stroke:#EA580C,stroke-width:2px,color:#111827;
    classDef dataflow fill:#D6E8FF,stroke:#2563EB,stroke-width:2px,color:#111827;
    classDef legend fill:#F3F4F6,stroke:#6B7280,stroke-width:1px,color:#111827;
    classDef header fill:#FFF4CC,stroke:#CA8A04,stroke-width:1px,color:#111827;

    class PM,NEWS,STOCK,PAPER,L1 external;
    class RAW,VALID,SHIFT,CONTEXT,IMPACT,SIGNAL,TRADE,STORED,DASH,L2 dataflow;
    class L3,L4,L5 legend;
    class GOAL header;
```

## Architecture Pattern and Justification

The system uses a combination of **pipeline** and **client-server** architecture. The pipeline pattern controls the main flow from collecting prediction market data to AI analysis, trading signals, and paper trading. The client-server pattern is used for the dashboard, which gets stored results from the backend.

| Criteria | Justification |
|---|---|
| **Fit to the Problem** | A pipeline fits because data moves through clear stages from market data to trading signals. Client-server fits the dashboard because it separates the user interface from the backend. |
| **Team Skills** | Both patterns are familiar and allow the system to be divided into components that can be worked on separately. |
| **Performance and Timing** | The pipeline processes new market data as it becomes available. The local AI model may be the slowest stage, so keeping it separate makes its performance easier to test. |
| **Scalability** | Individual components can be improved or expanded without changing the entire system. More data sources could also be added later. |
| **Hardware Constraints** | Qwen3.5 9B runs locally, so AI performance depends on available CPU, GPU, and memory. Keeping AI analysis separate limits its effect on the rest of the system. |

### Rejected Pattern

**Microservices** was considered but rejected because it would add unnecessary networking and deployment complexity for a two-person project. Pipeline and client-server provide enough separation without the extra infrastructure.

## Decision Log

| Decision | Alternatives Considered | Reason for Decision |
|---|---|---|
| Use **Polymarket** as the prediction market data source. | Polymarket, multiple prediction markets | Polymarket provides the prediction market data needed for the project while keeping the scope manageable. |
| Use **Qwen3.5 9B locally** for AI analysis. | Local Qwen3.5 9B, cloud-based AI API | Running the model locally avoids API costs and gives the team more control over the AI analysis. |
| Use **paper trading** to evaluate generated signals. | Paper trading, real-money trading, historical backtesting only | Paper trading allows the system to test signals using market conditions without risking real money. |
| Use a **pipeline and client-server architecture**. | Pipeline and client-server, microservices | Pipeline fits the step by step data processing, while client-server separates the dashboard from the backend without adding the complexity of microservices. |
| Store **market data, AI analysis, signals, and trade results**. | Store all analysis data, store only final trading results | Storing each stage makes it possible to trace how a trading signal was generated and evaluate previous results. |
