# DeFi Risk Lens — Development Roadmap

> Repository: `defi-risk-analyzer`
> Project: **DeFi Risk Lens**
> Document type: working product + engineering roadmap
> Status: target roadmap from empty repository to commercial MVP
> Audience: project owner / developer
>
> This roadmap is based on `README_implementation_blueprint.md` and earlier roadmap drafts.
> It is not a claim that all listed components are already implemented.

---

## 1. Roadmap objective

Build **DeFi Risk Lens** from an empty repository into a reproducible DeFi/on-chain risk analysis product.

The first working product should analyze:

- tokens;
- liquidity pools.

Later stages may add:

- protocol risk analysis;
- wallet/holder labeling;
- portfolio exposure analysis;
- dashboard;
- monitoring and alerts;
- hosted API / SaaS layer.

The product direction is **risk analysis**, not price prediction, trading signals, or automated trading.

The core positioning:

> We do not tell users what to buy. We show what can go wrong.

---

## 2. Non-goals

The project must not become:

- a price prediction tool;
- a trading-signal system;
- an automated trading bot;
- a wallet transaction signer;
- a guarantee that a token, pool, or protocol is safe;
- a guarantee of profitability;
- a full formal smart-contract audit;
- a tool that asks for or stores seed phrases, private keys, wallet passwords, or exchange passwords.

Correct output style:

> Available data indicates elevated risk.
> Confidence is limited because holder data is missing.
> Manual review is required before making any decision.

Avoid output style:

> This token is safe.
> This pool is guaranteed.
> This is definitely a scam.

---

## 3. Development principles

1. **Contracts before integrations**
   Implement schemas, validation, and deterministic demo fixtures before live APIs.

2. **Demo data before live data**
   The first end-to-end flow must work without API keys, rate limits, or RPC dependencies.

3. **Data provenance before scoring**
   Every metric must preserve source, timestamp, `chain_id`, and block/time range where applicable.

4. **Normalization before features**
   Chain, address, decimals, timestamps, units, and source status must be normalized before downstream feature calculation.

5. **Explainability before automation**
   A useful report with visible factors, confidence, and missing data is more important than an opaque automated score.

6. **Missing data is not low risk**
   Missing, stale, partial, or failed data must reduce confidence or produce `Insufficient data`.

7. **Reports before SaaS**
   Manual/demo reports should validate value before dashboard, monitoring, billing, or hosted API.

8. **Monitoring after repeatable checks**
   Alerts should only be implemented after token/pool analysis is stable, reproducible, and evidence-based.

9. **Narrow MVP first**
   Start with one or two chains, token/pool checks, demo fixtures, scoring v0, Markdown reports, and a simple local API.

---

## 4. Status legend

Use the same status language in roadmap, blueprint, issues, and task tracking.

| Status | Meaning |
|---|---|
| `PLANNED` | Target component is defined but not implemented. |
| `IN_PROGRESS` | Implementation started but not complete. |
| `IMPLEMENTED` | Component exists and passes minimal validation. |
| `PARTIAL` | Component exists but has known gaps. |
| `BLOCKED` | Cannot continue without missing input, source, or design decision. |
| `DEFERRED` | Deliberately postponed beyond MVP. |
| `REMOVED` | Explicitly removed from scope. |

---

## 5. Roadmap overview

| Phase | Name | Status | Main outcome |
|---:|---|---|---|
| 0 | Repository foundation | `PLANNED` | Public project structure, docs, safety boundaries |
| 1 | Package skeleton and configs | `PLANNED` | Importable Python package and basic config layout |
| 2 | Schemas and validation contracts | `PLANNED` | Stable data contracts before any scoring/API work |
| 3 | Demo fixtures and deterministic data source | `PLANNED` | Reproducible token/pool examples without network calls |
| 4 | Normalization layer | `PLANNED` | Chain, address, decimals, timestamps, source status |
| 5 | Token risk features v0 | `PLANNED` | First token risk factors from demo data |
| 6 | Liquidity pool features v0 | `PLANNED` | First pool/liquidity/APR risk factors |
| 7 | Explainable risk scoring v0 | `PLANNED` | Score, risk level, confidence, factor contributions |
| 8 | Markdown report generation | `PLANNED` | Human-readable token and pool demo reports |
| 9 | CLI/script entry point | `PLANNED` | One reproducible local demo run |
| 10 | FastAPI MVP | `PLANNED` | Local API with structured risk responses |
| 11 | Public demo assets and case studies | `PLANNED` | GitHub portfolio and outbound proof assets |
| 12 | Commercial validation with manual reports | `PLANNED` | Free mini-checks and first paid reports |
| 13 | External data source adapters | `DEFERRED` | Explorer/RPC/DEX/TVL adapters after demo pipeline |
| 14 | Dashboard MVP | `DEFERRED` | Visual risk review interface |
| 15 | Monitoring and alerts | `DEFERRED` | Risk deterioration alerts |
| 16 | Hosted API / SaaS evolution | `DEFERRED` | Paid API/monitoring after demand validation |

---

## 6. Phase 0 — Repository foundation

**Status:** `PLANNED`

### Goal

Create a professional repository foundation that explains the project, its boundaries, and its target development path.

### Deliverables

- `README.md`
- `README_architecture.md`
- `README_roadmap.md`
- `README_implementation_blueprint.md`
- `LICENSE` with Apache-2.0
- `.gitignore`
- `.env.example`
- `agents/`
- `optional_agents/`
- GitHub repository description

### Acceptance criteria

- The repository clearly says that DeFi Risk Lens is a risk analysis project, not a trading bot.
- The documentation separates planned architecture from implemented code.
- Financial-advice, profit-guarantee, and safety-guarantee wording is absent.
- A visitor can understand the project direction within a few minutes.

### Not in scope

- Real on-chain integration.
- Risk scoring implementation.
- API implementation.
- Dashboard.

### Validation

- Documentation review.
- Wording check for unsafe claims.
- Secret-handling check: `.env` must not be committed.

---

## 7. Phase 1 — Package skeleton and configs

**Status:** `PLANNED`

### Goal

Create an importable, testable Python project structure.

### Deliverables

- `pyproject.toml` or minimal project config.
- `src/defi_risk_analyzer/__init__.py`
- `configs/chains.yaml`
- `configs/sources.yaml`
- `configs/scoring.yaml`
- `configs/report_templates.yaml`
- `tests/`
- minimal package/version check.

### Acceptance criteria

- `from defi_risk_analyzer import __version__` works.
- Test command runs.
- Package name is consistent.
- `.env` and local secrets are ignored.

### Not in scope

- Feature extraction.
- Scoring.
- External API calls.

### Validation

- Import test.
- Minimal test run.
- `.gitignore` review.

---

## 8. Phase 2 — Schemas and validation contracts

**Status:** `PLANNED`

### Goal

Lock the project data contracts before building data logic, scoring, reports, or API.

### Deliverables

Implement schemas for:

- `AnalysisRequest`
- `ObjectIdentity`
- `SourceMetadata`
- `TokenMetadata`
- `PoolMetadata`
- `RiskFactor`
- `RiskResult`
- `MissingDataItem`

Implement validation for:

- `chain_id`;
- address format;
- known chain config;
- required source metadata;
- token decimals;
- risk result shape.

### Acceptance criteria

- Missing `chain_id` fails validation.
- Invalid address fails validation.
- Missing decimals blocks or reduces confidence according to contract.
- Missing source metadata fails for data-derived factors.
- `RiskResult` can never be score-only.

### Not in scope

- Live data connectors.
- Final scoring weights.
- Dashboard/API.

### Validation

- Schema tests.
- Serialization tests.
- Invalid input tests.
- Missing source metadata tests.

---

## 9. Phase 3 — Demo fixtures and deterministic data source

**Status:** `PLANNED`

### Goal

Build the first reproducible analysis flow without network calls or API keys.

### Deliverables

- `data/demo/token_example.json`
- `data/demo/pool_example.json`
- sample holder snapshot;
- sample liquidity snapshot;
- sample contract metadata;
- `data_sources/base.py`
- `data_sources/demo.py`
- expected output snapshot.

### Acceptance criteria

- One demo token loads.
- One demo pool loads.
- Every fixture includes `chain_id` and `SourceMetadata`.
- Failed fixture load returns explicit error, not low risk.
- Demo data can be used without API keys.

### Not in scope

- Live RPC/explorer integration.
- API rate limit logic.

### Validation

- Fixture load tests.
- Source metadata tests.
- Reproducibility snapshot test.

---

## 10. Phase 4 — Normalization layer

**Status:** `PLANNED`

### Goal

Normalize core fields before any downstream feature calculation.

### Deliverables

- address normalization;
- chain validation;
- token decimals and unit handling;
- timestamp normalization;
- source status handling;
- native-token vs ERC-20/BEP-20 distinction.

### Acceptance criteria

- The same address on different chains remains distinct.
- Normalized outputs preserve `chain_id`.
- Token decimals are not silently assumed.
- Partial or failed source status remains visible downstream.

### Not in scope

- Deep wallet labeling.
- Multi-chain portfolio aggregation.

### Validation

- Address normalization tests.
- Chain isolation tests.
- Decimals handling tests.
- Partial source status tests.

---

## 11. Phase 5 — Token risk features v0

**Status:** `PLANNED`

### Goal

Compute the first token-level risk factors from demo data.

### Deliverables

Feature modules for:

- token metadata;
- source verification flag;
- proxy/upgradeability flag;
- owner/admin presence;
- mint/burn permissions;
- blacklist/freeze/pause/transfer restriction flags;
- top-10/top-20 holder concentration;
- whale transfer count, if available;
- liquidity evidence status.

### Acceptance criteria

- Token analysis returns a structured list of `RiskFactor`.
- Each factor has category, value, evidence, source, risk level, and limitations.
- Contract risks and holder risks are separated.
- Missing holder or contract data reduces confidence.
- The output does not call a token safe or scam without evidence.

### Not in scope

- Formal smart-contract audit.
- Advanced bytecode analysis.
- Deep wallet clustering.
- CEX/router/pool wallet labeling as fact.

### Validation

- High holder concentration produces a high-risk factor.
- Missing holder snapshot becomes missing data.
- Proxy without implementation metadata is flagged or marked unknown.
- Unverified source affects risk/confidence.

---

## 12. Phase 6 — Liquidity pool features v0

**Status:** `PLANNED`

### Goal

Compute the first liquidity-pool risk factors from demo data.

### Deliverables

Feature modules for:

- pool identity;
- DEX and pool address;
- token pair metadata;
- TVL/liquidity depth;
- liquidity dynamics;
- LP ownership concentration;
- volume/liquidity ratio;
- slippage assumption;
- APR/APY realism;
- withdrawal/liquidity shock risk;
- dependency on token-level risks.

### Acceptance criteria

- Pool analysis returns liquidity, APR, and withdrawal-risk factors.
- TVL/APR values include source and timestamp.
- Low liquidity cannot produce high confidence.
- High APR is not treated as positive without sustainability checks.
- Missing LP ownership or APR source is visible.

### Not in scope

- Full DEX aggregator.
- Exact execution simulation.
- Production-grade slippage engine.
- Full reward-emission modeling.

### Validation

- Low TVL raises liquidity risk.
- High APR with low TVL raises sustainability risk.
- Missing APR source becomes unknown or reduces confidence.
- Partial liquidity history lowers confidence.

---

## 13. Phase 7 — Explainable risk scoring v0

**Status:** `PLANNED`

### Goal

Convert factor-level evidence into a transparent risk score and separate confidence score.

### Deliverables

- risk-level mapping;
- initial category weights;
- factor contribution calculation;
- confidence calculation;
- missing-data behavior;
- scoring config;
- known-case sanity tests.

### Initial risk levels

| Score | Level |
|---:|---|
| 0–24 | `Low risk` |
| 25–49 | `Medium risk` |
| 50–74 | `High risk` |
| 75–100 | `Critical risk` |
| N/A | `Insufficient data` |

### Suggested initial weights

| Category | Weight |
|---|---:|
| Contract red flags | 0.25 |
| Holder concentration | 0.20 |
| Liquidity risk | 0.25 |
| Market/volatility risk | 0.10 |
| APR/reward sustainability | 0.10 |
| Data quality | 0.10 |

These weights are placeholders and must be visible in output.

### Acceptance criteria

- Score is explainable at factor level.
- Weights are documented.
- Confidence is separate from score.
- Missing data reduces confidence.
- `Insufficient data` is a valid result.
- One metric cannot dominate silently.

### Not in scope

- Final calibrated model.
- ML scoring.
- Investment recommendations.

### Validation

- No factors -> `Insufficient data`.
- Missing critical data lowers confidence.
- Demo fixture returns stable expected result.
- High-risk fixture produces high-risk factors with evidence.
- Score-only output is impossible.

---

## 14. Phase 8 — Markdown report generation

**Status:** `PLANNED`

### Goal

Generate the first human-readable reports from structured risk results.

### Deliverables

- Markdown report template;
- report writer;
- raw metrics appendix;
- demo token report;
- demo liquidity pool report.

### Required report sections

```text
Executive summary
Object analyzed
Data sources
Final risk level
Risk score
Confidence
Main risk factors
Liquidity analysis
Holder concentration
Whale activity
Smart-contract red flags
APR / TVL sustainability
Missing data and limitations
Recommended next checks
Raw metrics appendix
```

### Acceptance criteria

- Report includes object identity.
- Report includes sources and timestamps.
- Report includes confidence.
- Report includes missing data and limitations.
- Report can be reused as public demo/case-study material.
- Report contains no investment-advice wording.

### Not in scope

- Confidential client reports in public repository.
- Legal/audit conclusions.
- PDF polish before Markdown works.

### Validation

- Required sections test.
- Wording safety test.
- Source metadata presence test.
- Snapshot test for demo reports.

---

## 15. Phase 9 — CLI/script entry point

**Status:** `PLANNED`

### Goal

Provide a simple local way to run the demo analysis end to end.

### Deliverables

- local script or CLI command;
- config-driven input;
- deterministic output path;
- generated demo report.

### Acceptance criteria

- Developer can run demo analysis from one command or script.
- No network is required for the first demo.
- Output is reproducible.
- Errors are explicit.

### Not in scope

- Production CLI.
- Live API ingestion.
- Complex command-line interface.

### Validation

- End-to-end demo run.
- Snapshot report comparison.
- Failure-mode test for missing fixture.

---

## 16. Phase 10 — FastAPI MVP

**Status:** `PLANNED`

### Goal

Expose the stable core logic through a simple local API.

### Deliverables

- `GET /health`
- `POST /v1/analyze/token`
- `POST /v1/analyze/pool`
- `POST /v1/report`
- API response schema;
- error schema;
- input validation tests;
- OpenAPI schema.

### Acceptance criteria

- API rejects invalid addresses and unsupported chains.
- Missing `chain_id` returns validation error.
- API returns factors, confidence, sources, missing data, and limitations.
- API failure does not return low risk by default.
- No secrets are logged.

### Not in scope

- Authentication.
- Billing.
- Public hosted production API.
- Live data source dependency.

### Validation

- Healthcheck test.
- Invalid address test.
- Missing `chain_id` test.
- Response shape test.
- Error schema test.
- No-secret logging review.

---

## 17. Phase 11 — Public demo assets and case studies

**Status:** `PLANNED`

### Goal

Turn the technical MVP into a credible portfolio and sales asset.

### Deliverables

- 3–5 demo reports;
- example API responses;
- case study pages;
- screenshots or report previews if useful;
- clear disclaimer;
- “What this project demonstrates” section.

### Acceptance criteria

- A potential client can see the deliverable before paying.
- Reports demonstrate liquidity, holders, contract, scoring, confidence, and missing-data logic.
- The repository supports outbound credibility.
- Demo reports do not expose client data or secrets.

### Not in scope

- Waiting for inbound clients from publications only.
- Guaranteed scam detection claims.
- Client-specific confidential reports.

### Validation

- Demo report review.
- Red-team wording review.
- Reproducibility check.

---

## 18. Phase 12 — Commercial validation with manual reports

**Status:** `PLANNED`

### Goal

Validate whether users will pay for DeFi risk analysis before building SaaS.

### First paid offer hypothesis

> Manual DeFi token/pool risk report with liquidity, holder concentration, whale activity, contract red flags, missing data, confidence, and recommended next checks.

### Pricing hypotheses

| Product | Initial price hypothesis | Deliverable |
|---|---:|---|
| Free mini-check | $0 | Short risk scan |
| Quick token risk report | $49–99 | Basic token report |
| Full token/pool report | $149–299 | Detailed risk report |
| Portfolio risk review | $199–499 | Multi-asset exposure review |
| Monthly monitoring | $99–299/month | Repeated checks and alerts |

These prices are hypotheses and must be validated.

### Deliverables

- free mini-check offer;
- paid report template;
- outbound message templates;
- client intake form;
- objection log;
- feedback loop into report/scoring methodology.

### Acceptance criteria

- 20–50 targeted outbound messages sent.
- At least 5 free mini-checks completed.
- At least 1 paid report sold or strong buying signal collected.
- Common objections are documented.
- Paid deliverable scope is clear.

### Not in scope

- Full SaaS billing.
- Large marketing campaigns.
- Vague “crypto analytics” positioning.

### Validation

- Outreach metrics.
- Conversion tracking.
- Client feedback review.
- Scope creep review.

---

## 19. Phase 13 — External data source adapters

**Status:** `DEFERRED` until demo pipeline is stable

### Goal

Add live data integrations only after contracts, fixtures, scoring, reports, and API shape are stable.

### Deliverables

- explorer adapter;
- RPC adapter;
- DEX/pool data adapter;
- TVL/APR source adapter;
- timeout/retry/rate-limit logic;
- raw data cache with provenance.

### Acceptance criteria

- One real token or pool can be analyzed with source timestamps and limitations.
- Pagination is handled.
- Partial failures do not become low risk.
- Raw data is cached with provenance.
- Data-source costs and rate limits are understood.

### Not in scope

- Broad multi-chain coverage.
- Expensive backfills.
- Production SLA.

### Validation

- Pagination test.
- Timeout test.
- Partial failure test.
- Source timestamp test.
- Raw cache provenance test.

---

## 20. Phase 14 — Dashboard MVP

**Status:** `DEFERRED`

### Goal

Create a visual interface for reviewing risk results after report/API flow is stable.

### Deliverables

- risk summary page;
- factor breakdown panel;
- confidence/data completeness panel;
- liquidity panel;
- holder concentration panel;
- contract red flags panel;
- report export button.

### Acceptance criteria

- Dashboard explains why a score is high or low.
- Missing data is visible.
- User can export or open a report.
- Score is never shown without factor evidence.

### Not in scope

- Advanced multi-user SaaS.
- Complex permissions.
- Billing.

### Validation

- UX review.
- Missing-data state review.
- Report export test.

---

## 21. Phase 15 — Monitoring and alerts

**Status:** `DEFERRED`

### Goal

Detect risk deterioration over time for selected tokens, pools, protocols, or portfolios.

### Deliverables

- watchlist model;
- alert rules;
- severity levels;
- deduplication logic;
- Telegram/email/webhook concept;
- alert history;
- missing-data behavior.

### Example triggers

- liquidity drop;
- TVL drop;
- top holder concentration increase;
- new whale transfer;
- suspicious mint or burn;
- admin/proxy change;
- APR anomaly;
- volume/liquidity anomaly;
- data-source failure.

### Acceptance criteria

- Every alert has object, source, timestamp, severity, threshold, and next action.
- Missing data can trigger warning or manual review.
- Alert deduplication prevents spam.
- API/source failure creates a data-source alert, not a low-risk output.

### Not in scope

- Automated trading actions.
- Wallet transaction signing.
- Liquidation or trading recommendations.

### Validation

- Threshold simulation.
- Deduplication test.
- Missing-data behavior test.
- Alert payload review.

---

## 22. Phase 16 — Hosted API / SaaS evolution

**Status:** `DEFERRED`

### Goal

Move from local tooling and manual reports to a hosted paid product only after demand is validated.

### Potential paid layers

- hosted risk API;
- monthly monitoring;
- portfolio exposure tracking;
- custom dashboards;
- premium data connectors;
- advanced scoring logic;
- team reports;
- enterprise monitoring.

### SaaS trigger conditions

Do not build SaaS until at least several are true:

- multiple users request repeated reports;
- manual report workflow becomes repetitive;
- users ask for monitoring;
- users ask for API access;
- at least one paying customer wants recurring checks;
- data-source costs and usage limits are understood.

### Not in scope before triggers

- billing system;
- account management;
- production multi-tenant dashboard;
- enterprise SLA.

---

## 23. Priority backlog

### P0 — Foundation

- Repository setup.
- Core README files.
- Apache-2.0 license.
- Implementation blueprint.
- Data contracts.
- Demo fixture plan.
- Report template.

### P1 — Reproducible technical MVP

- Package skeleton.
- Schemas.
- Validation.
- Demo fixtures.
- Normalization.
- Token risk factors.
- Pool risk factors.
- Risk scoring v0.
- Markdown reports.

### P2 — API and public demos

- CLI/script entry point.
- FastAPI endpoints.
- Example API responses.
- 3–5 demo reports.
- Case-study pages.

### P3 — Commercial validation

- Free mini-check offer.
- Paid report template.
- Outbound scripts.
- Client intake form.
- Feedback loop.
- First paid report.

### P4 — Productization

- External data adapters.
- Dashboard.
- Monitoring.
- Alerts.
- Hosted API.
- SaaS packaging.

---

## 24. Validation gates by roadmap stage

| Gate | Applies to | Required checks |
|---|---|---|
| Gate A — Data correctness | Phases 2–7, 13 | `chain_id`, address, decimals, source timestamp, block/time range, partial status |
| Gate B — Scoring correctness | Phase 7 | factors, weights, score logic, confidence logic, no score-only output |
| Gate C — Report correctness | Phase 8, 11, 12 | sources, confidence, missing data, limitations, no unsafe wording |
| Gate D — API correctness | Phase 10, 13 | validation errors, error schema, timeouts/retries, no low-risk on failure |
| Gate E — Security correctness | all phases | no secrets, no private keys, no unsafe signing, no unsafe logs |
| Gate F — Commercial validity | Phase 12+ | clear buyer, clear deliverable, outreach metrics, paid signal |

---

## 25. Stop conditions

Stop implementation and return to design review if:

- data from different chains is mixed without explicit `chain_id`;
- token decimals are missing but scoring continues confidently;
- source metadata is missing for data-derived metrics;
- API/source failure is treated as low risk;
- report shows final score without factor evidence;
- missing data is hidden;
- hardcoded secrets appear in code or examples;
- contract/pool is called safe without evidence;
- asset is called scam without explicit evidence;
- code attempts to sign or broadcast transactions;
- roadmap expands to broad multi-chain/SaaS before MVP is validated;
- dashboard, monitoring, or hosted SaaS work starts before reproducible demo reports exist.

---

## 26. Success metrics

### Technical metrics

- Number of implemented risk factors.
- Number of supported object types.
- Number of supported chains.
- Demo report reproducibility.
- API validation coverage.
- Missing-data visibility.
- Report generation success rate.
- Number of source-backed metrics.

### Commercial metrics

- Number of demo reports.
- Number of outbound messages sent.
- Reply rate.
- Free mini-check requests.
- Paid report conversions.
- Repeat monitoring requests.
- Requests for API/dashboard access.

---

## 27. Recommended 12-week development plan

This section turns the roadmap into a practical week-by-week development plan. It does not replace the roadmap phases. It maps them to calendar weeks: from an empty repository to a reproducible technical MVP and the first public demo assets.

Main principle: first build a reproducible local demo analysis without external APIs, then Markdown reports and API response shape, and only after that move to public demos, commercial validation, and live data adapters.

| Week | Focus | Main tasks | Weekly result | Readiness check |
|---:|---|---|---|---|
| 1 | Repository foundation | Finalize README files; review project positioning; add Apache-2.0 license; add .gitignore and .env.example; clearly separate risk analysis from trading, price prediction, and wallet transaction signing. | The repository looks like a professional DeFi/on-chain risk analysis project. | Documentation contains no profit guarantees, trading signals, safety guarantees, or unsupported scam claims; .env and secrets are not committed. |
| 2 | Package skeleton and configs | Create src/defi_risk_analyzer/; add __init__.py; add pyproject.toml or minimal project config; create configs/, data/demo/, reports/demo/, tests/; add configs/chains.yaml. | The project has an importable Python package structure. | defi_risk_analyzer can be imported; the test command runs; the package name is used consistently. |
| 3 | Core data schemas | Implement AnalysisRequest, ObjectIdentity, SourceMetadata, TokenMetadata, PoolMetadata, RiskFactor, RiskResult, MissingDataItem. | Core data contracts are fixed before features, scoring, reports, and API. | Schemas serialize to JSON; invalid or incomplete required fields fail predictably. |
| 4 | Validation layer | Implement validation for chain_id, address format, known chains, token decimals, source metadata, and source status. | Unsafe or ambiguous inputs cannot be silently accepted. | Missing chain_id is rejected; invalid address is rejected; missing source timestamp does not become low risk; missing decimals blocks analysis or lowers confidence according to the contract. |
| 5 | Demo fixtures | Add demo token fixture, demo pool fixture, holder snapshot, liquidity snapshot, contract metadata, and expected output snapshot. | The first reproducible data layer exists without network calls or API keys. | Fixtures load locally; every fixture contains chain_id and source metadata; fixture loading failure returns an explicit error, not low risk. |
| 6 | Data normalization | Implement address normalization, chain isolation, decimals/unit handling, timestamp normalization, source status propagation, and native token vs ERC-20/BEP-20-style token distinction. | Downstream modules receive normalized data. | The same address on different chains remains distinct; decimals are not silently assumed; partial/failed source status remains visible downstream. |
| 7 | Token risk features v0 | Implement token metadata features, source verification flag, proxy/admin indicators, mint/burn permissions, blacklist/freeze/pause/transfer restriction flags, top-10/top-20 holder concentration, and liquidity evidence status. | Demo token produces a structured list of risk factors. | Every factor has category, value, evidence, source, risk level, and limitations; missing holder/contract data lowers confidence. |
| 8 | Liquidity pool features v0 | Implement pool identity, DEX/pool address, token pair metadata, TVL/liquidity depth, liquidity dynamics, LP ownership concentration, volume/liquidity ratio, APR/APY realism, and withdrawal/liquidity shock risk. | Demo pool produces structured liquidity, APR, and withdrawal risk factors. | Low TVL raises liquidity risk; high APR with weak evidence raises sustainability risk; missing LP/APR data remains visible. |
| 9 | Explainable scoring v0 | Implement risk-level mapping, category weights, factor contributions, confidence calculation, missing-data behavior, and the Insufficient data result. | Token and pool results receive an explainable risk score and a separate confidence score. | Score-only output is impossible; missing critical data lowers confidence; a known demo fixture returns a stable expected result. |
| 10 | Markdown report generator | Create Markdown report template, report writer, raw metrics appendix; generate demo token report and demo pool report. | The first human-readable risk reports exist. | Reports include object identity, sources, timestamps, final risk level, score, confidence, factors, missing data, limitations, and recommended next checks. |
| 11 | Local run and API shape | Add a simple script/CLI for demo analysis; prepare deterministic output path; add example API response shape; start FastAPI skeleton with /health, /v1/analyze/token, /v1/analyze/pool. | Demo analysis can be run end to end locally; the API contract is clear. | One local demo run passes without network access; the API response shape includes factors, confidence, sources, missing data, and limitations. |
| 12 | Public demo assets | Prepare 3-5 demo reports, example API responses, case-study style README section, clear disclaimer, and a What this project demonstrates section. | The repository is ready as a proof asset for public review and early outreach. | Demo reports contain no client data or secrets; wording does not promise safety or profit; reports demonstrate liquidity, holders, contract, scoring, confidence, and missing-data logic. |

### Development priorities by week group

| Period | Main goal | What not to do during this period |
|---|---|---|
| Weeks 1-4 | Repository foundation, package skeleton, schemas, and validation. | Do not start live APIs, dashboard, monitoring, SaaS, or broad multi-chain support. |
| Weeks 5-8 | Demo fixtures, normalization, token features, and pool features. | Do not calibrate scoring on live assets before contracts and fixtures are stable. |
| Weeks 9-10 | Explainable scoring and Markdown reports. | Do not show a score without factors, confidence, missing data, and limitations. |
| Weeks 11-12 | Local run, API response shape, and public demo assets. | Do not move to hosted API/SaaS before commercial validation and repeated demand. |

### Readiness target after 12 weeks

By the end of week 12, the project should have a reproducible demo flow:

demo input -> normalized data -> risk factors -> score/confidence -> Markdown report -> example API response

Minimum successful output:

- object identity with chain_id and address;
- source metadata with timestamp and status;
- evidence for each risk factor;
- final risk level;
- risk score, if enough data is available;
- confidence;
- missing data;
- limitations;
- recommended next checks.

If this flow does not work end to end, external data adapters, dashboard, monitoring, and hosted SaaS remain DEFERRED.

---

## 28. Roadmap update policy

Update this roadmap when:

- a phase changes status;
- a schema changes;
- scoring logic changes;
- confidence logic changes;
- API response shape changes;
- report structure changes;
- external data adapters are introduced;
- a commercial assumption is validated or disproven;
- a deferred subsystem becomes active.

When implementation starts, each phase should be marked as one of:

- `PLANNED`
- `IN_PROGRESS`
- `IMPLEMENTED`
- `PARTIAL`
- `BLOCKED`
- `DEFERRED`
- `REMOVED`

Do not let this roadmap become a marketing document. Marketing positioning belongs in `README_marketing.md`. Public project explanation belongs in `README.md`. Implementation details belong in `README_implementation_blueprint.md`.
