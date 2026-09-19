# Constraints_Essay.md

**Project:** PolyPredict
**Team Members:** Vinesh Murikinati, Charlie Crocker

## Economic (Manufacturability)
Economic constraints directly dictate our software architectural decisions by restricting our total development budget to under $100 USD. To remain within this strict financial limit funded by personal student savings, our design relies exclusively on free-tier WebSockets from Polymarket and freeware Python libraries rather than paid enterprise feeds. This budget ceiling forces a major engineering trade off: satisfying our zero dollar data pipeline constraint means accepting lower API rate limits and higher latency, which restricts our viable solutions to lower frequency paper trading rather than high frequency execution.

## Legal (Regulatory)
Legal and regulatory frameworks govern how our system interacts with live financial markets and third-party data services. To strictly comply with SEC regulatory guidelines regarding algorithmic trading and the Alpaca API Terms of Use policy, our system is engineered exclusively for simulated paper trading environments with zero real capital deployment. Our primary user group retail paper traders and academic researchers requires explicit safeguards to prevent unintended market manipulation or illegal trade execution. Ensuring total adherence to API terms of service limits our trading engine to sandbox endpoints, preventing the deployment of live execution algorithms while keeping the solution fully compliant and risk free.

## Security
System security and data integrity shape how our engine processes real time news feeds and manages authentication credentials. Because our primary user group of retail paper traders relies on local execution, storing sensitive API keys for exchange access creates vulnerabilities to credential leakage or unauthorized web request manipulation. To prevent malicious API injection or unauthorized data tampering through our news scraping pipeline, all incoming WebSockets and REST payloads are validated through secure environment variable management and strict input sanitization protocols. Prioritizing strict local key security increases system setup complexity for end users, requiring manual configuration of secret environment files before the application can be safely deployed.
