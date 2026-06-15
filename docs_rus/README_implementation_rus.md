# DeFi Risk Lens — План реализации v2

> Репозиторий: `defi-risk-analyzer`  
> Проект: **DeFi Risk Lens**  
> Тип документа: внутренний инженерный план реализации  
> Статус: целевой план реализации. В репозитории код может ещё отсутствовать.  
> Аудитория: владелец проекта / разработчик.

Этот документ не является публичным GitHub README и не является маркетинговым документом.

Это рабочий инженерный blueprint для пошаговой разработки **DeFi Risk Lens**: модули, контракты, порядок реализации, статусы, validation gates, stop conditions и правила обновления.

---

## 1. Цель разработки

Построить объяснимый анализатор рисков DeFi/on-chain, который сначала оценивает:

- токены;
- пулы ликвидности.

В более поздних версиях можно добавить:

- анализ риска протоколов;
- маркировку wallet/holder;
- анализ portfolio exposure;
- dashboards;
- monitoring and alerts;
- hosted API/SaaS layer.

Система должна отвечать:

- Какие видимые risk factors есть у этого токена или пула?
- Какие features увеличили или снизили risk score?
- Какие данные отсутствуют, устарели, частичны или ненадёжны?
- Насколько высока confidence результата?
- Что нужно проверить вручную перед принятием любого решения?

Проект не должен предоставлять:

- прогнозы цены;
- торговые сигналы;
- автоматическую торговлю;
- подписание транзакций кошелька;
- инвестиционные рекомендации;
- гарантии безопасности или прибыльности;
- неподтверждённые утверждения, что актив точно является scam.

Корректный стиль вывода:

> Доступные данные указывают на повышенный риск.  
> Confidence ограничена, потому что отсутствуют данные по holders.  
> Перед принятием любого решения требуется ручная проверка.

---

## 2. Граница реализации MVP

Первая реализация должна быть намеренно узкой.

### Scope MVP

Построить:

- один Python package: `defi_risk_analyzer`;
- одну или две поддерживаемые сети, явно настроенные через `chain_id`;
- deterministic demo fixtures до live APIs;
- token analysis v0;
- liquidity pool analysis v0;
- source metadata and data provenance;
- explainable risk scoring v0;
- confidence and missing-data logic;
- генерацию Markdown report;
- простой FastAPI API после стабилизации core logic;
- небольшие demo reports.

### Не входит в MVP

Пока не строить:

- широкое multi-chain coverage;
- полный protocol analysis;
- portfolio analysis;
- live monitoring;
- dashboard;
- billing/SaaS layer;
- ML scoring;
- automatic wallet transaction signing;
- полноценный smart-contract audit engine;
- deep wallet clustering.

### Путь разработки

Безопасная последовательность реализации:

```text
contracts -> demo data -> normalization -> features -> scoring -> report -> API -> public demos
```

Не начинать с live data, dashboard, monitoring или SaaS behavior.

---

## 3. Легенда статусов реализации

Использовать эти статусы в этом документе и в task planning.

| Status | Значение |
|---|---|
| `PLANNED` | Целевой компонент определён, но не реализован. |
| `IN_PROGRESS` | Реализация начата, но не завершена. |
| `IMPLEMENTED` | Компонент существует и проходит минимальную валидацию. |
| `PARTIAL` | Компонент существует, но имеет известные gaps. |
| `BLOCKED` | Нельзя продолжать без недостающего input, source или design decision. |
| `DEFERRED` | Намеренно отложено за пределы MVP. |
| `REMOVED` | Явно удалено из scope. |

Текущий глобальный статус:

| Область | Статус | Примечания |
|---|---|---|
| Repository foundation | `PLANNED` | Package skeleton не предполагается реализованным |
| Package skeleton | `PLANNED` | `src/defi_risk_analyzer` не предполагается реализованным |
| Data contracts | `PLANNED` | Требуются до scoring/API |
| Demo fixtures | `PLANNED` | Использовать до live APIs |
| Normalization | `PLANNED` | Address, chain, decimals, timestamps |
| Token risk module | `PLANNED` | Contract + holders + whales v0 |
| Liquidity pool module | `PLANNED` | TVL/liquidity/APR/slippage v0 |
| Risk scoring v0 | `PLANNED` | Explainable score + confidence |
| Markdown report generation | `PLANNED` | Первый human-readable output |
| FastAPI schema | `PLANNED` | После стабилизации core logic |
| Dashboard | `DEFERRED` | Не MVP |
| Monitoring/alerts | `DEFERRED` | Не MVP |
| SaaS/API commercial layer | `DEFERRED` | После demo reports and demand validation |

---

## 4. Защищённые инженерные контракты

Эти правила обязательны для всего проекта.

### 4.1 Identity contract

Каждый анализируемый on-chain объект должен включать:

```json
{
  "object_type": "token | pool | protocol | wallet | portfolio",
  "chain_id": 1,
  "address": "0x...",
  "name": "optional",
  "symbol": "optional"
}
```

Правила:

- `chain_id` обязателен.
- Один и тот же address в разных сетях не является одним и тем же объектом.
- Addresses должны нормализоваться.
- Checksummed address form должна сохраняться там, где применимо.
- Native tokens и ERC-20/BEP-20-style tokens должны обрабатываться отдельно.

### 4.2 Source metadata contract

Каждая метрика должна быть трассируема до источника.

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

Правила:

- нет метрики без source metadata;
- нет freshness-sensitive result без `checked_at`;
- нет block-sensitive result без block или time range;
- `status=partial` должен добавлять limitation;
- `status=failed` не должен порождать low-risk factor;
- API failures не должны превращаться в low-risk values.

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

Блокирующие проблемы:

- отсутствует `chain_id`;
- отсутствуют или предполагаются `decimals`;
- source unverified, когда требуется contract logic;
- proxy contract без проверки implementation/admin;
- stale или unknown source timestamp.

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

Блокирующие проблемы:

- отсутствует `chain_id`;
- pair tokens не разрешены;
- отсутствуют token decimals;
- отсутствует source metadata;
- unknown DEX/pool type, когда интерпретация liquidity зависит от него.

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

Правила:

- каждый factor должен иметь category;
- каждый non-unknown factor должен иметь evidence;
- missing evidence должен делать factor risk `Unknown` или снижать confidence;
- ни один factor не может заявлять safety;
- factor modules не вычисляют final score.

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

Правила:

- API/report никогда не должен возвращать только score.
- Missing data должны снижать confidence, а не скрывать risk.
- `Insufficient data` — валидный result, а не exception.
- Report/API output должен включать factors, sources, missing data, confidence и limitations.

---

## 5. Целевая структура репозитория

Создавать эту структуру постепенно. Не создавать пустые сложные подсистемы до того, как они понадобятся.

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

## 6. Карта core modules

| Модуль | Путь | Статус | Назначение | MVP? |
|---|---|---:|---|---:|
| Config loading | `src/defi_risk_analyzer/config/` | `PLANNED` | Загрузка chain/source/scoring/report configs. | Yes |
| Schemas | `src/defi_risk_analyzer/schemas/` | `PLANNED` | Data contracts для requests, sources, features, scores, reports. | Yes |
| Validation | `validation/` | `PLANNED` | Validate chain ID, address format, object type, required fields. | Yes |
| Data sources | `data_sources/` | `PLANNED` | Adapters для file/demo/API/RPC sources. | Yes |
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

## 7. Data contracts, которые нужно реализовать первыми

Реализация должна начинаться с contracts до внешних API integrations.

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

Правила валидации:

- `object_type` должен быть explicit.
- `chain_id` required для on-chain objects.
- `address` должен быть validated для target chain format.
- `analysis_depth` defaults to `quick` for MVP.
- unknown chain не должен проходить, если не configured.

### 7.2 `SourceMetadata`

Использовать contract из section 4.2.

Обязательная валидация:

- `checked_at` required;
- failed source не должен генерировать low risk;
- partial source должен добавлять limitations;
- missing block/time range снижает confidence.

### 7.3 `TokenMetadata`

Использовать contract из section 4.3.

Обязательная валидация:

- `decimals` должны иметь source;
- `source_verified` должен быть explicit;
- proxy metadata нельзя игнорировать.

### 7.4 `PoolMetadata`

Использовать contract из section 4.4.

Обязательная валидация:

- pair tokens должны быть resolved;
- decimals обоих tokens должны быть known;
- source timestamp required.

### 7.5 `RiskFactor`

Использовать contract из section 4.5.

Обязательная валидация:

- category required;
- evidence required для non-unknown factors;
- source required для data-derived factors.

### 7.6 `RiskResult`

Использовать contract из section 4.6.

Обязательная валидация:

- object identity present;
- risk level present;
- confidence present;
- factors present;
- missing data visible.

---

## 8. План реализации features

### 8.1 Token risk features — MVP

| Feature | Статус | Требуемый input | Output | Validation |
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

| Feature | Статус | Требуемый input | Output | Validation |
|---|---|---|---|---|
| Pool identity | `PLANNED` | chain_id, pool address, DEX, pair | `PoolMetadata` | token pair resolved |
| TVL/liquidity depth | `PLANNED` | DEX/API/demo data | factor | source timestamp required |
| Liquidity dynamics | `PLANNED` | historical liquidity | factor | partial history -> lower confidence |
| LP ownership concentration | `PLANNED` | LP holder data | factor | missing LP owners -> missing data |
| Volume/liquidity ratio | `PLANNED` | volume, liquidity | factor | timestamp consistency required |
| Slippage assumption | `PLANNED` | liquidity depth, trade size | factor | clearly mark assumptions |
| APR/APY realism | `PLANNED` | APR source, rewards, TVL | factor | no APR source -> unknown |

### 8.3 Holder and whale features — partial MVP

| Feature | Статус | Требуемый input | Output | Validation |
|---|---|---|---|---|
| Holder count | `PLANNED` | holder snapshot | factor | block/time required |
| Top-10/top-20 share | `PLANNED` | holder snapshot | factor | block/time required |
| Whale transfer count | `PLANNED` | transfer events | factor | event range required |
| Wallet labels | `DEFERRED` | label source/manual list | assumptions | do not state clustering as fact |
| CEX/router/pool wallet detection | `DEFERRED` | labels/contracts | limitations | unknown labels reduce confidence |

### 8.4 Contract red flags — basic MVP

| Feature | Статус | Требуемый input | Output | Validation |
|---|---|---|---|---|
| Source verified | `PLANNED` | explorer/source/demo metadata | factor | source timestamp required |
| Proxy/admin | `PLANNED` | proxy metadata | factor | implementation required if proxy |
| Privileged methods | `PLANNED` | ABI/source scan/demo list | factor | list method names as evidence |
| Transfer restrictions | `PLANNED` | ABI/source scan/demo list | factor | manual review if uncertain |

Этот module не является formal smart-contract audit.

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

Это starting scale. Позже её нужно откалибровать на real examples.

### 9.2 Suggested MVP category weights

Initial weights являются placeholders и должны быть visible in output.

| Category | Weight | Использование в MVP |
|---|---:|---|
| Contract red flags | 0.25 | proxy, owner/admin, mint, blacklist, pause, tax |
| Holder concentration | 0.20 | top holders, whale concentration |
| Liquidity risk | 0.25 | TVL, liquidity depth, LP concentration, liquidity drop |
| Market/volatility risk | 0.10 | volatility, volume anomaly, drawdown if available |
| APR/reward sustainability | 0.10 | APR realism, reward token dependency |
| Data quality | 0.10 | stale, missing, partial, failed sources |

Правила:

- missing data не должны создавать low risk;
- missing data должны снижать confidence;
- category with unknown critical features должна быть marked as limited confidence;
- feature contributions должны включаться в report/API output;
- одна metric не должна silently dominate.

### 9.3 Confidence v0

Confidence начинается с `1.0` и снижается при таких проблемах:

| Issue | Рекомендуемый штраф |
|---|---:|
| Missing contract source | -0.20 |
| Missing holder snapshot | -0.15 |
| Missing liquidity history | -0.15 |
| Missing decimals/source metadata | blocking or -0.25 |
| Partial API pagination | -0.10 |
| Stale source timestamp | -0.10 |
| Unknown block/time range | -0.10 |
| Wallet labels inferred, not verified | -0.05 to -0.15 |

Если confidence падает ниже заданного threshold, возвращать `Insufficient data` или clearly limited conclusion, а не confident score.

---

## 10. Report contract

Первый report format должен быть Markdown. HTML/PDF можно добавить позже.

Обязательные разделы:

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

Правила формулировок report:

- не обещать safety;
- не обещать profitability;
- не предоставлять trading signals;
- не называть asset confirmed scam без explicit evidence;
- не скрывать missing data;
- не представлять unsupported wallet labels как facts.

---

## 11. API contract — MVP

API не требуется до scoring/report MVP, но schemas должны быть API-ready.

Планируемые endpoints:

```text
GET  /health
POST /v1/analyze/token
POST /v1/analyze/pool
POST /v1/report
```

Каждый response должен включать:

- analyzed object;
- source metadata;
- risk level;
- risk score, if available;
- confidence;
- factor list;
- missing data;
- limitations;
- recommended next checks.

Запрещённое поведение API:

- возвращать только score;
- silently принимать invalid address;
- трактовать API failure как low risk;
- логировать API keys;
- не использовать timeout для external calls.

---

## 12. Порядок реализации

Не начинать с external API integrations. Начать с contracts, fixtures и deterministic demo analysis.

### Этап 0 — Основа репозитория

Статус: `PLANNED`

Результаты:

- package skeleton;
- `.gitignore`;
- `.env.example`;
- `pyproject.toml` or minimal project config;
- `configs/` directory;
- `data/demo/` directory;
- `tests/` directory.

Проверка:

- repository imports cleanly;
- secrets не committed;
- package name is consistent.

Критерии готовности:

- `from defi_risk_analyzer import __version__` works;
- test command runs;
- no `.env` or API keys are committed.

### Этап 1 — Schemas and configs

Статус: `PLANNED`

Результаты:

- `AnalysisRequest`;
- `SourceMetadata`;
- `TokenMetadata`;
- `PoolMetadata`;
- `RiskFactor`;
- `RiskResult`;
- `chains.yaml`;
- `scoring.yaml`;
- `sources.yaml`.

Проверка:

- schema validation tests;
- invalid chain/address tests;
- missing decimals test;
- missing source metadata test.

Критерии готовности:

- schemas serialize to JSON;
- invalid inputs fail predictably;
- risk response contract is stable.

### Этап 2 — Demo fixtures

Статус: `PLANNED`

Результаты:

- static demo token fixture;
- static demo pool fixture;
- sample holder snapshot;
- sample liquidity snapshot;
- sample contract metadata;
- expected output snapshot.

Проверка:

- fixtures load without network;
- all fixture data has `chain_id` and source metadata;
- expected output is reproducible.

Критерии готовности:

- один demo token loads;
- один demo pool loads;
- failed fixture load returns explicit error, not low risk.

### Этап 3 — Слой нормализации

Статус: `PLANNED`

Результаты:

- address normalization;
- chain validation;
- decimals handling;
- timestamp normalization;
- source status handling.

Проверка:

- same address on different chains remains distinct;
- missing decimals blocks or lowers confidence;
- partial source stays partial.

Критерии готовности:

- normalized outputs preserve `chain_id`;
- no silent decimals assumption exists.

### Этап 4 — Feature extraction v0

Статус: `PLANNED`

Результаты:

- token metadata features;
- holder concentration features;
- basic contract red flags;
- liquidity depth features;
- missing-data feature records.

Проверка:

- every feature has source metadata;
- unknown features become missing data;
- factor categories are stable.

Критерии готовности:

- demo token produces token factors;
- demo pool produces liquidity factors;
- no asset is called safe.

### Этап 5 — Risk scoring v0

Статус: `PLANNED`

Результаты:

- risk-level mapping;
- category weights;
- factor contributions;
- confidence calculation;
- `Insufficient data` behavior.

Проверка:

- score-only output is impossible;
- missing data reduces confidence;
- known fixture returns stable expected result;
- high-risk fixture produces high-risk factors with evidence.

Критерии готовности:

- demo token and demo pool have risk score, confidence, factors, missing data, and limitations.

### Этап 6 — Генерация Markdown report

Статус: `PLANNED`

Результаты:

- Markdown report template;
- report writer;
- demo report in `reports/demo/`;
- raw metrics appendix.

Проверка:

- report includes sources;
- report includes confidence;
- report includes missing data;
- report contains no investment-advice wording.

Критерии готовности:

- at least two demo reports exist:
  - one token report;
  - one liquidity pool report.

### Этап 7 — CLI или script entry point

Статус: `PLANNED`

Результаты:

- simple local script for demo analysis;
- config-driven input;
- output report path.

Проверка:

- command works on demo fixture;
- no network required for first demo;
- output is reproducible.

Критерии готовности:

- developer can run demo analysis locally from a single command or script.

### Этап 8 — FastAPI skeleton

Статус: `PLANNED`

Результаты:

- `/health`;
- `/v1/analyze/token`;
- `/v1/analyze/pool`;
- API response schema;
- error schema.

Проверка:

- invalid address returns validation error;
- missing `chain_id` returns validation error;
- response includes factors and confidence;
- no secrets in logs.

Критерии готовности:

- FastAPI app runs locally;
- OpenAPI schema is generated;
- demo token and pool can be analyzed through API.

### Этап 9 — Адаптеры внешних data sources

Status: `DEFERRED` until demo pipeline is stable.

Результаты:

- explorer adapter;
- RPC adapter;
- DEX/pool data adapter;
- TVL/APR source adapter;
- retry/timeout/rate-limit logic.

Проверка:

- pagination handled;
- source timestamp stored;
- partial failures do not become low risk;
- raw data cached with provenance.

Критерии готовности:

- one real token or pool can be analyzed with source timestamps and limitations.

### Этап 10 — Dashboard and monitoring

Status: `DEFERRED`.

Результаты:

- dashboard panels;
- alert rules;
- deduplication;
- alert severity;
- source/block/timestamp in every alert.

Проверка:

- missing data states visible;
- alerts do not fire repeatedly without deduplication;
- API failure creates data-source alert, not low-risk output.

Критерии готовности:

- later-stage only; not required for MVP completion.

---

## 13. Validation gates

Использовать эти gates перед тем, как помечать phase как `IMPLEMENTED`.

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

Остановить implementation и вернуться к redesign, если происходит любое из следующего:

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

1. Создать package skeleton and directories.
2. Добавить `.gitignore` and `.env.example`.
3. Добавить `configs/chains.yaml` with 1–2 chains.
4. Добавить schema classes for object identity, source metadata, token metadata, pool metadata, risk factor, and risk result.
5. Добавить validation tests for `chain_id`, address, decimals, and missing source timestamp.
6. Добавить demo fixtures for one token and one pool.
7. Добавить deterministic feature extractor from demo fixtures.
8. Добавить scoring v0 with visible factor contributions.
9. Добавить Markdown report generator.
10. Добавить demo report and snapshot test.

Не начинать с full external API ingestion, dashboard, monitoring или SaaS billing.

---

## 16. Developer workflow

Для каждой implementation task:

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

Обновлять этот blueprint, когда:

- a module is added;
- a schema changes;
- scoring logic changes;
- confidence logic changes;
- API response changes;
- report structure changes;
- a phase moves from `PLANNED` to `IN_PROGRESS`, `PARTIAL`, or `IMPLEMENTED`;
- a deferred subsystem becomes active.

Не превращать этот документ в marketing document. Public positioning belongs in `README.md` or `README_marketing.md`.

---

## 18. Final developer summary

Первый успешный milestone проекта — не красивый UI.

Первый успешный milestone — воспроизводимый demo analysis, где output включает:

- object identity;
- source metadata;
- factor-level risk evidence;
- final risk level;
- score;
- confidence;
- missing data;
- limitations;
- recommended next checks.

Строить в этом порядке:

```text
contracts -> demo fixtures -> normalization -> features -> scoring -> markdown report -> API -> public demos
```

Всё остальное вторично, пока этот путь не работает end to end.
