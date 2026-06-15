# DeFi Risk Lens

**DeFi Risk Lens** is an explainable DeFi and on-chain risk analysis project for tokens, liquidity pools, protocols, and crypto portfolios.

The repository name is:

```text
defi-risk-analyzer
```

The goal of this project is not to predict prices, generate trading signals, or promise profit.

The goal is to make DeFi risk more visible, structured, and evidence-based.

## What this project analyzes

DeFi Risk Lens focuses on risk factors such as:

* liquidity depth and liquidity changes;
* holder concentration;
* whale activity;
* suspicious transfers;
* smart-contract red flags;
* proxy and admin control risks;
* mint, burn, blacklist, pause, and transfer-restriction logic;
* TVL and APR sustainability;
* volatility and market risk;
* protocol dependencies;
* missing data and confidence limitations.

## Core principle

A risk score should never be a magic number.

Every risk result should explain:

* which features were used;
* what increased or reduced the risk;
* what data sources were used;
* what data is missing;
* how confident the analysis is;
* what should be checked manually next.

## Features

Planned and/or in-progress features:

* token risk review;
* liquidity pool risk review;
* protocol risk summary;
* holder concentration analysis;
* whale activity summary;
* smart-contract red-flag checklist;
* explainable risk scoring;
* confidence and missing-data reporting;
* HTML/PDF-style risk report structure;
* API-ready risk response format;
* future monitoring and alerting rules.

More detailed feature documentation will be added later.

## Intended output

The project is designed to produce structured risk outputs such as:

* token risk reviews;
* liquidity pool risk reviews;
* protocol risk summaries;
* holder concentration analysis;
* whale activity summaries;
* smart-contract red-flag checks;
* HTML/PDF risk reports;
* API-ready risk responses;
* future monitoring and alerting rules.

## Risk levels

The intended risk levels are:

* `Low risk`
* `Medium risk`
* `High risk`
* `Critical risk`
* `Insufficient data`

If the available data is incomplete, stale, or unreliable, the correct result is not a confident score. It is reduced confidence or `Insufficient data`.

## Example report

Example reports will be added later.

Planned report structure:

```text
Executive summary
Object analyzed
Data sources
Final risk level
Risk score
Confidence
Liquidity analysis
Holder concentration
Whale activity
Smart-contract red flags
Volatility and market risk
APR / TVL sustainability
Key risks
Missing data and limitations
Recommended next checks
Raw metrics appendix
```

## API example

API examples will be added later.

Planned response shape:

```json
{
  "object": {
    "type": "token_or_pool",
    "chain_id": 1,
    "address": "0x..."
  },
  "risk": {
    "level": "High risk",
    "score": 78,
    "confidence": 0.72
  },
  "factors": [
    {
      "name": "top_holders_concentration",
      "value": "67%",
      "risk": "High",
      "comment": "Top holders control a large share of supply."
    },
    {
      "name": "liquidity_depth",
      "value": "$180,000",
      "risk": "Medium",
      "comment": "Liquidity may be insufficient for larger exits."
    }
  ],
  "missing_data": [
    "verified_contract_source",
    "historical_lp_ownership"
  ],
  "data_sources": [
    {
      "name": "to_be_defined",
      "checked_at": "to_be_defined"
    }
  ]
}
```

## Roadmap

### Phase 1 — Project foundation

* Define project structure.
* Define risk report format.
* Define risk scoring principles.
* Add demo data.
* Add first static example reports.

### Phase 2 — Token and pool analysis

* Implement token metadata extraction.
* Add holder concentration checks.
* Add liquidity pool metrics.
* Add basic contract red-flag checks.
* Add missing-data and confidence logic.

### Phase 3 — Explainable risk scoring

* Define feature weights.
* Add factor-level contributions.
* Add confidence score.
* Add validation examples.
* Add known-case sanity checks.

### Phase 4 — Reports and API

* Generate structured HTML/PDF-style reports.
* Add API response schema.
* Add simple FastAPI endpoint.
* Add example requests and responses.

### Phase 5 — Monitoring and commercial layer

* Add alerting rules.
* Add liquidity and whale monitoring.
* Add Telegram/email notification concept.
* Add paid risk report workflow.
* Add hosted API/SaaS roadmap.

## What this project is not

This project is **not**:

* financial advice;
* a trading bot;
* a price prediction tool;
* a guarantee that a token, pool, or protocol is safe;
* a tool for automatically buying, selling, or signing transactions;
* a full formal smart-contract audit.

The project may identify risk indicators, but it does not guarantee that an asset is safe or unsafe.

## Safety and data handling

This project must never request or store:

* seed phrases;
* private keys;
* wallet passwords;
* exchange passwords;
* hardcoded API secrets.

All analysis should be based on public on-chain data, public protocol data, user-provided non-sensitive data, or safe API integrations.

## Commercial direction

DeFi Risk Lens is designed as both:

* an open technical portfolio project;
* a foundation for paid DeFi risk reports, monitoring, and API-based risk analysis.

The public repository demonstrates the methodology and engineering approach.

Commercial value may come from:

* fresh data;
* hosted infrastructure;
* monitoring;
* custom reports;
* integrations;
* support;
* expert interpretation.

## License

This project is licensed under the **Apache License 2.0**.

See the `LICENSE` file for details.

## Disclaimer

This project is for educational, analytical, and risk-research purposes only.

It does not provide investment advice, financial recommendations, trading signals, or guarantees of safety or profitability.

Always perform independent due diligence before interacting with any DeFi asset, pool, protocol, or smart contract.
