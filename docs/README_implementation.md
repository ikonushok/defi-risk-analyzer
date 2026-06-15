# DeFi Risk Lens — Implementation Blueprint v2

> Repository: `defi-risk-analyzer`  
> Project: **DeFi Risk Lens**  
> Document type: internal developer implementation blueprint  
> Status: target implementation plan. The repository may not yet contain code.  
> Audience: project owner / developer.

This document is not a public GitHub README and not a marketing document.

It is a working engineering blueprint for building **DeFi Risk Lens** step by step: modules, contracts, implementation order, statuses, validation gates, stop conditions, and update rules.

---

## 1. Development objective

Build an explainable DeFi/on-chain risk analyzer that first evaluates:

- tokens;
- liquidity pools.

Later versions may add:

- protocol risk analysis;
- wallet/holder labeling;
- portfolio exposure analysis;
- dashboards;
- monitoring and alerts;
- hosted API/SaaS layer.

The system should answer:

- What visible risk factors exist for this token or pool?
- Which features increased or reduced the risk score?
- Which data is missing, stale, partial, or unreliable?
- How confident is the result?
- What should be checked manually before making any decision?

The project must not provide:

- price predictions;
- trading signals;
- automated trading;
- wallet transaction signing;
- investment recommendations;
- guarantees of safety or profitability;
- unsupported claims that an asset is definitely a scam.

Correct output style:

> Available data indicates elevated risk.  
> Confidence is limited because holder data is missing.  
> Manual review is required before making any decision.

---

## 2. MVP implementation boundary

The first implementation must be intentionally narrow.

### MVP scope

Build:

- one Python package: `defi_risk_analyzer`;
- one or two supported chains, configured explicitly by `chain_id`;
- deterministic demo fixtures before live APIs;
- token analysis v0;
- liquidity pool analysis v0;
- source metadata and data provenance;
- explainable risk scoring v0;
- confidence and missing-data logic;
- Markdown report generation;
- simple FastAPI API after core logic is stable;
- small demo reports.

### Not in MVP

Do not build yet:

- broad multi-chain coverage;
- full protocol analysis;
- portfolio analysis;
- live monitoring;
- dashboard;
- billing/SaaS layer;
- ML scoring;
- automatic wallet transaction signing;
- full smart-contract audit engine;
- deep wallet clustering.

### Developer path

The safe implementation sequence is:

```text
contracts -> demo data -> normalization -> features -> scoring -> report -> API -> public demos
```

Do not start with live data, dashboard, monitoring, or SaaS behavior.

---

## 3. Implementation status legend

Use these statuses in this document and in task planning.

| Status | Meaning |
|---|---|
| `PLANNED` | Target component is defined but not implemented. |
| `IN_PROGRESS` | Implementation started but not complete. |
| `IMPLEMENTED` | Component exists and passes minimal validation. |
| `PARTIAL` | Component exists but has known gaps. |
| `BLOCKED` | Cannot continue without missing input, source, or design decision. |
| `DEFERRED` | Deliberately postponed beyond MVP. |
| `REMOVED` | Explicitly removed from scope. |

Current global status:

| Area | Status | Notes |
|---|---|---|
| Repository foundation | `PLANNED` | Package skeleton not assumed implemented |
| Package skeleton | `PLANNED` | `src/defi_risk_analyzer` not assumed implemented |
| Data contracts | `PLANNED` | Required before scoring/API |
| Demo fixtures | `PLANNED` | Use before live APIs |
| Normalization | `PLANNED` | Address, chain, decimals, timestamps |
| Token risk module | `PLANNED` | Contract + holders + whales v0 |
| Liquidity pool module | `PLANNED` | TVL/liquidity/APR/slippage v0 |
| Risk scoring v0 | `PLANNED` | Explainable score + confidence |
| Markdown report generation | `PLANNED` | First human-readable output |
| FastAPI schema | `PLANNED` | After core logic is stable |
| Dashboard | `DEFERRED` | Not MVP |
| Monitoring/alerts | `DEFERRED` | Not MVP |
| SaaS/API commercial layer | `DEFERRED` | After demo reports and demand validation |

---

## 4. Protected engineering contracts

These rules are mandatory across the project.

### 4.1 Identity contract

Every analyzed on-chain object must include:

```json
{
  "object_type": "token | pool | protocol | wallet | portfolio",
  "chain_id": 1,
  "address": "0x...",
  "name": "optional",
  "symbol": "optional"
}
```

Rules:

- `chain_id` is mandatory.
- The same address on different chains is not the same object.
- Addresses must be normalized.
- Checksummed address form should be preserved where applicable.
- Native tokens and ERC-20/BEP-20-style tokens must be handled separately.

### 4.2 Source metadata contract

Every metric must be traceable to a source.

```json
{
  "source_name": "etherscan | rpc | dex_api | defillama | user_file | demo_fixture",
  "source_type": "api | rpc | file | manual | fixture",
  "checked_at": "2026-06-15T00:00:00Z",
  "chain_id": 1,
  "block_number": 12345678,
  "time_range": {
    "from": "optional",
    "to": "optional"
  },
  "status": "ok | partial | failed",
  "is_partial": false,
  "limitations": []
}
```

Rules:

- no metric without source metadata;
- no freshness-sensitive result without `checked_at`;
- no block-sensitive result without block or time range;
- `status=partial` must add a limitation;
- `status=failed` must not produce a low-risk factor;
- API failures must not become low-risk values.

### 4.3 Token metadata contract

```json
{
  "chain_id": 1,
  "address": "0x...",
  "symbol": "TOKEN",
  "name": "Token Name",
  "decimals": 18,
  "total_supply": "1000000000000000000000000",
  "source_verified": true,
  "is_proxy": false,
  "implementation_address": null,
  "source": {}
}
```

Blocking issues:

- missing `chain_id`;
- missing or assumed `decimals`;
- unverified source when contract logic is required;
- proxy contract without implementation/admin review;
- stale or unknown source timestamp.

### 4.4 Pool metadata contract

```json
{
  "chain_id": 1,
  "pool_address": "0x...",
  "dex": "uniswap_v2_or_v3_or_other",
  "token0": {
    "address": "0x...",
    "symbol": "TOKEN0",
    "decimals": 18
  },
  "token1": {
    "address": "0x...",
    "symbol": "TOKEN1",
    "decimals": 18
  },
  "source": {}
}
```

Blocking issues:

- missing `chain_id`;
- unresolved pair tokens;
- missing token decimals;
- missing source metadata;
- unknown DEX/pool type when liquidity interpretation depends on it.

### 4.5 Risk factor contract

```json
{
  "name": "top_holders_concentration",
  "category": "holders",
  "value": "67%",
  "risk_level": "High",
  "weight": 0.2,
  "contribution": 16,
  "evidence": "Top holders control a large share of supply.",
  "source": {},
  "limitations": []
}
```

Rules:

- every factor must have a category;
- every non-unknown factor must have evidence;
- missing evidence should make factor risk `Unknown` or reduce confidence;
- no factor can claim safety;
- factor modules do not compute final score.

### 4.6 Risk result contract

```json
{
  "object": {
    "object_type": "token",
    "chain_id": 1,
    "address": "0x..."
  },
  "risk": {
    "level": "High risk",
    "score": 78,
    "confidence": 0.72,
    "score_version": "v0.1"
  },
  "factors": [],
  "missing_data": [],
  "limitations": [],
  "recommended_next_checks": []
}
```

Rules:

- API/report must never return only a score.
- Missing data must reduce confidence, not hide risk.
- `Insufficient data` is a valid result, not an exception.
- Report/API output must include factors, sources, missing data, confidence, and limitations.

---

## 5. Target repository structure

Create this structure incrementally. Do not create empty complex subsystems before they are needed.

```text
defi-risk-analyzer/
├── README.md
├── README_architecture.md
├── README_roadmap.md
├── README_marketing.md
├── README_implementation_blueprint.md
├── LICENSE
├── pyproject.toml
├── .gitignore
├── .env.example
│
├── configs/
│   ├── chains.yaml
│   ├── sources.yaml
│   ├── scoring.yaml
│   └── report_templates.yaml
│
├── data/
│   ├── raw/
│   ├── processed/
│   ├── features/
│   └── demo/
│
├── examples/
│   ├── inputs/
│   ├── reports/
│   └── api_responses/
│
├── reports/
│   └── demo/
│
├── src/
│   └── defi_risk_analyzer/
│       ├── __init__.py
│       ├── config/
│       │   ├── chains.py
│       │   ├── sources.py
│       │   └── scoring.py
│       ├── schemas/
│       │   ├── common.py
│       │   ├── sources.py
│       │   ├── token.py
│       │   ├── pool.py
│       │   ├── features.py
│       │   └── risk.py
│       ├── validation/
│       │   ├── addresses.py
│       │   └── chains.py
│       ├── data_sources/
│       │   ├── base.py
│       │   ├── demo.py
│       │   └── cache.py
│       ├── normalization/
│       │   ├── addresses.py
│       │   ├── units.py
│       │   └── timestamps.py
│       ├── features/
│       │   ├── token_contract.py
│       │   ├── holders.py
│       │   ├── liquidity.py
│       │   ├── whales.py
│       │   ├── market.py
│       │   └── apr.py
│       ├── scoring/
│       │   ├── levels.py
│       │   ├── weights.py
│       │   ├── confidence.py
│       │   └── engine.py
│       ├── reports/
│       │   ├── markdown.py
│       │   └── templates/
│       ├── api/
│       │   ├── main.py
│       │   ├── routes.py
│       │   └── schemas.py
│       ├── monitoring/
│       │   └── rules.py
│       └── utils/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
│
├── agents/
└── optional_agents/
```

---

## 6. Core module map

| Module | Path | Status | Purpose | MVP? |
|---|---|---:|---|---:|
| Config loading | `src/defi_risk_analyzer/config/` | `PLANNED` | Load chain/source/scoring/report configs. | Yes |
| Schemas | `src/defi_risk_analyzer/schemas/` | `PLANNED` | Data contracts for requests, sources, features, scores, reports. | Yes |
| Validation | `validation/` | `PLANNED` | Validate chain ID, address format, object type, required fields. | Yes |
| Data sources | `data_sources/` | `PLANNED` | Adapters for file/demo/API/RPC sources. | Yes |
| Normalization | `normalization/` | `PLANNED` | Address, chain, decimals, timestamp, units normalization. | Yes |
| Token contract features | `features/token_contract.py` | `PLANNED` | Contract source, proxy, owner/admin, mint, blacklist, pause flags. | Yes |
| Holder features | `features/holders.py` | `PLANNED` | Top holders, concentration, holder snapshot quality. | Yes |
| Whale features | `features/whales.py` | `PLANNED` | Whale events from transfers, if available. | Partial |
| Liquidity features | `features/liquidity.py` | `PLANNED` | TVL, liquidity depth, LP concentration, liquidity dynamics. | Yes |
| Market features | `features/market.py` | `PLANNED` | Volatility, drawdown, abnormal volume. | Later MVP |
| APR features | `features/apr.py` | `PLANNED` | APR source and reward sustainability warnings. | Partial |
| Scoring | `scoring/` | `PLANNED` | Risk levels, score v0, confidence, factor contributions. | Yes |
| Reports | `reports/` | `PLANNED` | Markdown first; HTML/PDF later. | Yes |
| API | `api/` | `PLANNED` | FastAPI endpoints and response schemas. | Basic MVP |
| Monitoring | `monitoring/` | `DEFERRED` | Alert rules, thresholds, deduplication, notifications. | No |
| Tests | `tests/` | `PLANNED` | Unit and validation checks. | Yes |

---

## 7. Data contracts to implement first

Implementation should start from contracts before external API integrations.

### 7.1 `AnalysisRequest`

```json
{
  "object_type": "token | pool | protocol | wallet | portfolio",
  "chain_id": 1,
  "address": "0x...",
  "time_range": {
    "from": null,
    "to": null
  },
  "analysis_depth": "quick | standard | full"
}
```

Validation rules:

- `object_type` must be explicit.
- `chain_id` is required for on-chain objects.
- `address` must be validated for the target chain format.
- `analysis_depth` defaults to `quick` for MVP.
- unknown chain must not pass unless configured.

### 7.2 `SourceMetadata`

Use the contract from section 4.2.

Required validation:

- `checked_at` is required;
- failed source must not generate low risk;
- partial source must add limitations;
- missing block/time range reduces confidence.

### 7.3 `TokenMetadata`

Use the contract from section 4.3.

Required validation:

- `decimals` must be sourced;
- `source_verified` must be explicit;
- proxy metadata must not be ignored.

### 7.4 `PoolMetadata`

Use the contract from section 4.4.

Required validation:

- pair tokens must be resolved;
- both token decimals must be known;
- source timestamp is required.

### 7.5 `RiskFactor`

Use the contract from section 4.5.

Required validation:

- category required;
- evidence required for non-unknown factors;
- source required for data-derived factors.

### 7.6 `RiskResult`

Use the contract from section 4.6.

Required validation:

- object identity present;
- risk level present;
- confidence present;
- factors present;
- missing data visible.

---

## 8. Feature implementation plan

### 8.1 Token risk features — MVP

| Feature | Status | Required input | Output | Validation |
|---|---|---|---|---|
| Token metadata | `PLANNED` | chain_id, address, decimals, symbol, supply | `TokenMetadata` | decimals sourced, chain_id present |
| Source verification flag | `PLANNED` | explorer/source/demo metadata | factor | unverified source -> confidence penalty |
| Proxy flag | `PLANNED` | source/bytecode/demo metadata | factor | proxy without implementation -> missing data |
| Admin/owner presence | `PLANNED` | ABI/source/manual/demo metadata | factor | owner unknown -> missing data |
| Mint/burn permissions | `PLANNED` | ABI/source/manual/demo metadata | factor | privileged mint -> risk factor |
| Blacklist/freeze/pause flags | `PLANNED` | ABI/source/manual/demo metadata | factor | detected restriction -> risk factor |
| Top-10 holders share | `PLANNED` | holder snapshot | factor | snapshot must include block/time |
| Top-20 holders share | `PLANNED` | holder snapshot | factor | snapshot must include block/time |
| Liquidity evidence | `PLANNED` | pool/liquidity source | factor | missing liquidity -> confidence penalty |

### 8.2 Liquidity pool features — MVP

| Feature | Status | Required input | Output | Validation |
|---|---|---|---|---|
| Pool identity | `PLANNED` | chain_id, pool address, DEX, pair | `PoolMetadata` | token pair resolved |
| TVL/liquidity depth | `PLANNED` | DEX/API/demo data | factor | source timestamp required |
| Liquidity dynamics | `PLANNED` | historical liquidity | factor | partial history -> lower confidence |
| LP ownership concentration | `PLANNED` | LP holder data | factor | missing LP owners -> missing data |
| Volume/liquidity ratio | `PLANNED` | volume, liquidity | factor | timestamp consistency required |
| Slippage assumption | `PLANNED` | liquidity depth, trade size | factor | clearly mark assumptions |
| APR/APY realism | `PLANNED` | APR source, rewards, TVL | factor | no APR source -> unknown |

### 8.3 Holder and whale features — partial MVP

| Feature | Status | Required input | Output | Validation |
|---|---|---|---|---|
| Holder count | `PLANNED` | holder snapshot | factor | block/time required |
| Top-10/top-20 share | `PLANNED` | holder snapshot | factor | block/time required |
| Whale transfer count | `PLANNED` | transfer events | factor | event range required |
| Wallet labels | `DEFERRED` | label source/manual list | assumptions | do not state clustering as fact |
| CEX/router/pool wallet detection | `DEFERRED` | labels/contracts | limitations | unknown labels reduce confidence |

### 8.4 Contract red flags — basic MVP

| Feature | Status | Required input | Output | Validation |
|---|---|---|---|---|
| Source verified | `PLANNED` | explorer/source/demo metadata | factor | source timestamp required |
| Proxy/admin | `PLANNED` | proxy metadata | factor | implementation required if proxy |
| Privileged methods | `PLANNED` | ABI/source scan/demo list | factor | list method names as evidence |
| Transfer restrictions | `PLANNED` | ABI/source scan/demo list | factor | manual review if uncertain |

This module is not a formal smart-contract audit.

---

## 9. Scoring v0 design

### 9.1 Risk levels

| Score | Level |
|---:|---|
| 0–24 | `Low risk` |
| 25–49 | `Medium risk` |
| 50–74 | `High risk` |
| 75–100 | `Critical risk` |
| N/A | `Insufficient data` |

This is a starting scale. It must be calibrated later with real examples.

### 9.2 Suggested MVP category weights

Initial weights are placeholders and must be visible in output.

| Category | Weight | MVP use |
|---|---:|---|
| Contract red flags | 0.25 | proxy, owner/admin, mint, blacklist, pause, tax |
| Holder concentration | 0.20 | top holders, whale concentration |
| Liquidity risk | 0.25 | TVL, liquidity depth, LP concentration, liquidity drop |
| Market/volatility risk | 0.10 | volatility, volume anomaly, drawdown if available |
| APR/reward sustainability | 0.10 | APR realism, reward token dependency |
| Data quality | 0.10 | stale, missing, partial, failed sources |

Rules:

- missing data should not create low risk;
- missing data should reduce confidence;
- a category with unknown critical features should be marked as limited confidence;
- feature contributions must be included in report/API output;
- one metric must not dominate silently.

### 9.3 Confidence v0

Confidence starts at `1.0` and decreases for issues such as:

| Issue | Suggested penalty |
|---|---:|
| Missing contract source | -0.20 |
| Missing holder snapshot | -0.15 |
| Missing liquidity history | -0.15 |
| Missing decimals/source metadata | blocking or -0.25 |
| Partial API pagination | -0.10 |
| Stale source timestamp | -0.10 |
| Unknown block/time range | -0.10 |
| Wallet labels inferred, not verified | -0.05 to -0.15 |

If confidence falls below a defined threshold, return `Insufficient data` or a clearly limited conclusion, not a confident score.

---

## 10. Report contract

The first report format should be Markdown. HTML/PDF can come later.

Required sections:

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

Report wording rules:

- do not promise safety;
- do not promise profitability;
- do not provide trading signals;
- do not call an asset a confirmed scam without explicit evidence;
- do not hide missing data;
- do not present unsupported wallet labels as facts.

---

## 11. API contract — MVP

API is not required before the scoring/report MVP, but schemas should be API-ready.

Planned endpoints:

```text
GET  /health
POST /v1/analyze/token
POST /v1/analyze/pool
POST /v1/report
```

Every response must include:

- analyzed object;
- source metadata;
- risk level;
- risk score, if available;
- confidence;
- factor list;
- missing data;
- limitations;
- recommended next checks.

Forbidden API behavior:

- return only score;
- accept invalid address silently;
- treat API failure as low risk;
- log API keys;
- use no timeout around external calls.

---

## 12. Implementation order

Do not start with external API integrations. Start with contracts, fixtures, and deterministic demo analysis.

### Phase 0 — Repository foundation

Status: `PLANNED`

Deliverables:

- package skeleton;
- `.gitignore`;
- `.env.example`;
- `pyproject.toml` or minimal project config;
- `configs/` directory;
- `data/demo/` directory;
- `tests/` directory.

Validation:

- repository imports cleanly;
- no secrets committed;
- package name is consistent.

Definition of done:

- `from defi_risk_analyzer import __version__` works;
- test command runs;
- no `.env` or API keys are committed.

### Phase 1 — Schemas and configs

Status: `PLANNED`

Deliverables:

- `AnalysisRequest`;
- `SourceMetadata`;
- `TokenMetadata`;
- `PoolMetadata`;
- `RiskFactor`;
- `RiskResult`;
- `chains.yaml`;
- `scoring.yaml`;
- `sources.yaml`.

Validation:

- schema validation tests;
- invalid chain/address tests;
- missing decimals test;
- missing source metadata test.

Definition of done:

- schemas serialize to JSON;
- invalid inputs fail predictably;
- risk response contract is stable.

### Phase 2 — Demo fixtures

Status: `PLANNED`

Deliverables:

- static demo token fixture;
- static demo pool fixture;
- sample holder snapshot;
- sample liquidity snapshot;
- sample contract metadata;
- expected output snapshot.

Validation:

- fixtures load without network;
- all fixture data has `chain_id` and source metadata;
- expected output is reproducible.

Definition of done:

- one demo token loads;
- one demo pool loads;
- failed fixture load returns explicit error, not low risk.

### Phase 3 — Normalization layer

Status: `PLANNED`

Deliverables:

- address normalization;
- chain validation;
- decimals handling;
- timestamp normalization;
- source status handling.

Validation:

- same address on different chains remains distinct;
- missing decimals blocks or lowers confidence;
- partial source stays partial.

Definition of done:

- normalized outputs preserve `chain_id`;
- no silent decimals assumption exists.

### Phase 4 — Feature extraction v0

Status: `PLANNED`

Deliverables:

- token metadata features;
- holder concentration features;
- basic contract red flags;
- liquidity depth features;
- missing-data feature records.

Validation:

- every feature has source metadata;
- unknown features become missing data;
- factor categories are stable.

Definition of done:

- demo token produces token factors;
- demo pool produces liquidity factors;
- no asset is called safe.

### Phase 5 — Risk scoring v0

Status: `PLANNED`

Deliverables:

- risk-level mapping;
- category weights;
- factor contributions;
- confidence calculation;
- `Insufficient data` behavior.

Validation:

- score-only output is impossible;
- missing data reduces confidence;
- known fixture returns stable expected result;
- high-risk fixture produces high-risk factors with evidence.

Definition of done:

- demo token and demo pool have risk score, confidence, factors, missing data, and limitations.

### Phase 6 — Markdown report generation

Status: `PLANNED`

Deliverables:

- Markdown report template;
- report writer;
- demo report in `reports/demo/`;
- raw metrics appendix.

Validation:

- report includes sources;
- report includes confidence;
- report includes missing data;
- report contains no investment-advice wording.

Definition of done:

- at least two demo reports exist:
  - one token report;
  - one liquidity pool report.

### Phase 7 — CLI or script entry point

Status: `PLANNED`

Deliverables:

- simple local script for demo analysis;
- config-driven input;
- output report path.

Validation:

- command works on demo fixture;
- no network required for first demo;
- output is reproducible.

Definition of done:

- developer can run demo analysis locally from a single command or script.

### Phase 8 — FastAPI skeleton

Status: `PLANNED`

Deliverables:

- `/health`;
- `/v1/analyze/token`;
- `/v1/analyze/pool`;
- API response schema;
- error schema.

Validation:

- invalid address returns validation error;
- missing `chain_id` returns validation error;
- response includes factors and confidence;
- no secrets in logs.

Definition of done:

- FastAPI app runs locally;
- OpenAPI schema is generated;
- demo token and pool can be analyzed through API.

### Phase 9 — External data source adapters

Status: `DEFERRED` until demo pipeline is stable.

Deliverables:

- explorer adapter;
- RPC adapter;
- DEX/pool data adapter;
- TVL/APR source adapter;
- retry/timeout/rate-limit logic.

Validation:

- pagination handled;
- source timestamp stored;
- partial failures do not become low risk;
- raw data cached with provenance.

Definition of done:

- one real token or pool can be analyzed with source timestamps and limitations.

### Phase 10 — Dashboard and monitoring

Status: `DEFERRED`.

Deliverables:

- dashboard panels;
- alert rules;
- deduplication;
- alert severity;
- source/block/timestamp in every alert.

Validation:

- missing data states visible;
- alerts do not fire repeatedly without deduplication;
- API failure creates data-source alert, not low-risk output.

Definition of done:

- later-stage only; not required for MVP completion.

---

## 13. Validation gates

Use these gates before marking a phase as `IMPLEMENTED`.

### Gate A — Data correctness

Required:

- `chain_id` present;
- address valid;
- decimals sourced;
- source timestamp present;
- block/time range recorded;
- partial source status represented;
- raw data not overwritten without provenance.

### Gate B — Scoring correctness

Required:

- factor list present;
- score formula/logic documented;
- weights documented;
- confidence logic tested;
- missing data reduces confidence;
- no score-only output.

### Gate C — Report correctness

Required:

- object identity visible;
- sources visible;
- risk score and confidence visible;
- missing data visible;
- limitations visible;
- recommended next checks present;
- no safety/profit/scam overclaims.

### Gate D — API correctness

Required:

- input validation;
- structured error schema;
- timeout/retry behavior for external calls;
- no secrets in logs;
- response includes factors and confidence;
- failed source does not produce low risk.

### Gate E — Security correctness

Required:

- `.env` ignored;
- no hardcoded API keys;
- no private key/seed phrase handling;
- no wallet transaction signing;
- no unsafe shell/eval on untrusted input.

---

## 14. Stop conditions

Stop implementation and redesign if any of these occur:

- data from different chains is mixed without explicit `chain_id`;
- token decimals are missing but scoring continues as confident;
- API failure is treated as low risk;
- report shows final score without factor evidence;
- missing data is hidden;
- hardcoded secrets appear in code;
- contract/pool is called safe without evidence;
- asset is called scam without explicit evidence;
- code attempts to sign or broadcast transactions.

---

## 15. First practical coding tasks

Recommended first 10 implementation tasks:

1. Create package skeleton and directories.
2. Add `.gitignore` and `.env.example`.
3. Add `configs/chains.yaml` with 1–2 chains.
4. Add schema classes for object identity, source metadata, token metadata, pool metadata, risk factor, and risk result.
5. Add validation tests for `chain_id`, address, decimals, and missing source timestamp.
6. Add demo fixtures for one token and one pool.
7. Add deterministic feature extractor from demo fixtures.
8. Add scoring v0 with visible factor contributions.
9. Add Markdown report generator.
10. Add demo report and snapshot test.

Do not start with full external API ingestion, dashboard, monitoring, or SaaS billing.

---

## 16. Developer workflow

For each implementation task:

1. Inspect existing files before editing.
2. Define the smallest affected module.
3. Update or add one contract/test first when possible.
4. Implement minimal local logic.
5. Run the narrowest validation.
6. Record what was not validated.
7. Update status in this blueprint when a phase changes.

Suggested task template:

```markdown
## Goal

## Affected modules

## Contracts at risk

## Minimal change

## Validation command

## Expected result

## What is not validated
```

---

## 17. Document update rules

Update this blueprint when:

- a module is added;
- a schema changes;
- scoring logic changes;
- confidence logic changes;
- API response changes;
- report structure changes;
- a phase moves from `PLANNED` to `IN_PROGRESS`, `PARTIAL`, or `IMPLEMENTED`;
- a deferred subsystem becomes active.

Do not let this document become a marketing document. Public positioning belongs in `README.md` or `README_marketing.md`.

---

## 18. Final developer summary

The first successful project milestone is not a beautiful UI.

The first successful milestone is a reproducible demo analysis where the output includes:

- object identity;
- source metadata;
- factor-level risk evidence;
- final risk level;
- score;
- confidence;
- missing data;
- limitations;
- recommended next checks.

Build in this order:

```text
contracts -> demo fixtures -> normalization -> features -> scoring -> markdown report -> API -> public demos
```

Everything else is secondary until this path works end to end.
