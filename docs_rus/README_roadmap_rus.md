# DeFi Risk Lens — Roadmap разработки

> Репозиторий: `defi-risk-analyzer`  
> Проект: **DeFi Risk Lens**  
> Тип документа: рабочий product + engineering roadmap  
> Статус: целевой roadmap от пустого репозитория до commercial MVP  
> Аудитория: владелец проекта / разработчик  
>
> Этот roadmap основан на `README_implementation_blueprint.md` и более ранних roadmap drafts.  
> Он не является утверждением, что все перечисленные компоненты уже реализованы.

---

## 1. Цель roadmap

Построить **DeFi Risk Lens** из пустого репозитория в воспроизводимый DeFi/on-chain risk analysis product.

Первый рабочий продукт должен анализировать:

- токены;
- пулы ликвидности.

На следующих этапах можно добавить:

- анализ риска протоколов;
- маркировку wallet/holder;
- анализ portfolio exposure;
- dashboard;
- monitoring and alerts;
- hosted API / SaaS layer.

Product direction — **risk analysis**, а не price prediction, trading signals или automated trading.

Core positioning:

> Мы не говорим пользователям, что покупать. Мы показываем, что может пойти не так.

---

## 2. Не входит в цели

Проект не должен превращаться в:

- инструмент price prediction;
- систему trading signals;
- automated trading bot;
- wallet transaction signer;
- гарантию, что token, pool или protocol safe;
- гарантию profitability;
- полноценный formal smart-contract audit;
- инструмент, который запрашивает или хранит seed phrases, private keys, wallet passwords или exchange passwords.

Корректный стиль вывода:

> Доступные данные указывают на повышенный риск.  
> Confidence ограничена, потому что отсутствуют данные по holders.  
> Перед принятием любого решения требуется ручная проверка.

Избегать такого стиля вывода:

> Этот токен безопасен.  
> Этот пул гарантирован.  
> Это точно scam.

---

## 3. Принципы разработки

1. **Contracts before integrations**  
   Реализовать schemas, validation и deterministic demo fixtures до live APIs.

2. **Demo data before live data**  
   Первый end-to-end flow должен работать без API keys, rate limits или RPC dependencies.

3. **Data provenance before scoring**  
   Каждая metric должна сохранять source, timestamp, `chain_id` и block/time range, где применимо.

4. **Normalization before features**  
   Chain, address, decimals, timestamps, units и source status должны быть normalized до downstream feature calculation.

5. **Explainability before automation**  
   Полезный report с visible factors, confidence и missing data важнее, чем opaque automated score.

6. **Missing data is not low risk**  
   Missing, stale, partial или failed data должны снижать confidence или давать `Insufficient data`.

7. **Reports before SaaS**  
   Manual/demo reports должны validated value до dashboard, monitoring, billing или hosted API.

8. **Monitoring after repeatable checks**  
   Alerts следует реализовывать только после того, как token/pool analysis stable, reproducible и evidence-based.

9. **Narrow MVP first**  
   Начать с одной или двух chains, token/pool checks, demo fixtures, scoring v0, Markdown reports и простого local API.

---

## 4. Легенда статусов

Использовать одинаковые status language в roadmap, blueprint, issues и task tracking.

| Status | Значение |
|---|---|
| `PLANNED` | Целевой компонент определён, но не реализован. |
| `IN_PROGRESS` | Реализация начата, но не завершена. |
| `IMPLEMENTED` | Компонент существует и проходит минимальную валидацию. |
| `PARTIAL` | Компонент существует, но имеет известные gaps. |
| `BLOCKED` | Нельзя продолжать без недостающего input, source или design decision. |
| `DEFERRED` | Намеренно отложено за пределы MVP. |
| `REMOVED` | Явно удалено из scope. |

---

## 5. Обзор roadmap

| Фаза | Название | Статус | Главный результат |
|---:|---|---|---|
| 0 | Основа репозитория | `PLANNED` | Публичная структура проекта, документация, safety boundaries |
| 1 | Скелет пакета и конфиги | `PLANNED` | Импортируемый Python package и базовый config layout |
| 2 | Schemas и validation contracts | `PLANNED` | Стабильные data contracts до scoring/API |
| 3 | Demo fixtures и deterministic data source | `PLANNED` | Воспроизводимые примеры token/pool без network calls |
| 4 | Слой нормализации | `PLANNED` | Chain, address, decimals, timestamps, source status |
| 5 | Token risk features v0 | `PLANNED` | Первые token risk factors из demo data |
| 6 | Liquidity pool features v0 | `PLANNED` | Первые pool/liquidity/APR risk factors |
| 7 | Explainable risk scoring v0 | `PLANNED` | Score, risk level, confidence, factor contributions |
| 8 | Генерация Markdown report | `PLANNED` | Human-readable token and pool demo reports |
| 9 | CLI/script entry point | `PLANNED` | Один reproducible local demo run |
| 10 | FastAPI MVP | `PLANNED` | Local API со structured risk responses |
| 11 | Public demo assets and case studies | `PLANNED` | GitHub portfolio и outbound proof assets |
| 12 | Коммерческая валидация manual reports | `PLANNED` | Free mini-checks и первые paid reports |
| 13 | Адаптеры external data sources | `DEFERRED` | Explorer/RPC/DEX/TVL adapters после demo pipeline |
| 14 | Dashboard MVP | `DEFERRED` | Visual risk review interface |
| 15 | Monitoring and alerts | `DEFERRED` | Risk deterioration alerts |
| 16 | Hosted API / SaaS evolution | `DEFERRED` | Paid API/monitoring после demand validation |

---

## 6. Этап 0 — Основа репозитория

**Статус:** `PLANNED`

### Цель

Создать professional repository foundation, который объясняет project, его boundaries и target development path.

### Результаты

- `README.md`
- `README_architecture.md`
- `README_roadmap.md`
- `README_implementation_blueprint.md`
- `LICENSE` с Apache-2.0
- `.gitignore`
- `.env.example`
- `agents/`
- `optional_agents/`
- описание GitHub repository

### Критерии приёмки

- Репозиторий ясно говорит, что DeFi Risk Lens — это risk analysis project, а не trading bot.
- Документация отделяет planned architecture от implemented code.
- Financial-advice, profit-guarantee и safety-guarantee wording отсутствуют.
- Посетитель может понять направление проекта за несколько минут.

### Не входит в scope

- Real on-chain integration.
- Risk scoring implementation.
- API implementation.
- Dashboard.

### Проверка

- Review документации.
- Проверка формулировок на unsafe claims.
- Проверка secret-handling: `.env` не должен быть committed.

---

## 7. Этап 1 — Скелет пакета и конфиги

**Статус:** `PLANNED`

### Цель

Создать importable, testable Python project structure.

### Результаты

- `pyproject.toml` или минимальный project config.
- `src/defi_risk_analyzer/__init__.py`
- `configs/chains.yaml`
- `configs/sources.yaml`
- `configs/scoring.yaml`
- `configs/report_templates.yaml`
- `tests/`
- минимальная package/version check.

### Критерии приёмки

- `from defi_risk_analyzer import __version__` works.
- Test command runs.
- Package name is consistent.
- `.env` и local secrets игнорируются.

### Не входит в scope

- Feature extraction.
- Scoring.
- External API calls.

### Проверка

- Import test.
- Minimal test run.
- Review `.gitignore`.

---

## 8. Этап 2 — Schemas и validation contracts

**Статус:** `PLANNED`

### Цель

Зафиксировать project data contracts до построения data logic, scoring, reports или API.

### Результаты

Реализовать schemas для:

- `AnalysisRequest`
- `ObjectIdentity`
- `SourceMetadata`
- `TokenMetadata`
- `PoolMetadata`
- `RiskFactor`
- `RiskResult`
- `MissingDataItem`

Реализовать validation для:

- `chain_id`;
- address format;
- known chain config;
- required source metadata;
- token decimals;
- risk result shape.

### Критерии приёмки

- Missing `chain_id` fails validation.
- Invalid address fails validation.
- Missing decimals блокирует анализ или снижает confidence согласно contract.
- Missing source metadata приводит к ошибке для data-derived factors.
- `RiskResult` никогда не может быть score-only.

### Не входит в scope

- Live data connectors.
- Final scoring weights.
- Dashboard/API.

### Проверка

- Schema tests.
- Serialization tests.
- Invalid input tests.
- Missing source metadata tests.

---

## 9. Этап 3 — Demo fixtures и deterministic data source

**Статус:** `PLANNED`

### Цель

Построить первый reproducible analysis flow без network calls или API keys.

### Результаты

- `data/demo/token_example.json`
- `data/demo/pool_example.json`
- sample holder snapshot;
- sample liquidity snapshot;
- sample contract metadata;
- `data_sources/base.py`
- `data_sources/demo.py`
- expected output snapshot.

### Критерии приёмки

- Один demo token loads.
- Один demo pool loads.
- Каждый fixture включает `chain_id` и `SourceMetadata`.
- Ошибка загрузки fixture возвращает explicit error, а не low risk.
- Demo data можно использовать без API keys.

### Не входит в scope

- Live RPC/explorer integration.
- API rate limit logic.

### Проверка

- Fixture load tests.
- Source metadata tests.
- Reproducibility snapshot test.

---

## 10. Этап 4 — Слой нормализации

**Статус:** `PLANNED`

### Цель

Нормализовать core fields до любого downstream feature calculation.

### Результаты

- address normalization;
- chain validation;
- token decimals and unit handling;
- timestamp normalization;
- source status handling;
- distinction native-token vs ERC-20/BEP-20.

### Критерии приёмки

- Один и тот же address в разных chains остаётся distinct.
- Normalized outputs сохраняют `chain_id`.
- Token decimals не предполагаются silently.
- Partial или failed source status остаётся visible downstream.

### Не входит в scope

- Deep wallet labeling.
- Multi-chain portfolio aggregation.

### Проверка

- Address normalization tests.
- Chain isolation tests.
- Decimals handling tests.
- Partial source status tests.

---

## 11. Этап 5 — Token risk features v0

**Статус:** `PLANNED`

### Цель

Вычислить первые token-level risk factors from demo data.

### Результаты

Feature modules для:

- token metadata;
- source verification flag;
- proxy/upgradeability flag;
- owner/admin presence;
- mint/burn permissions;
- blacklist/freeze/pause/transfer restriction flags;
- top-10/top-20 holder concentration;
- whale transfer count, if available;
- liquidity evidence status.

### Критерии приёмки

- Token analysis returns a structured list of `RiskFactor`.
- Each factor has category, value, evidence, source, risk level, and limitations.
- Contract risks and holder risks are separated.
- Missing holder or contract data reduces confidence.
- The output does not call a token safe or scam without evidence.

### Не входит в scope

- Formal smart-contract audit.
- Advanced bytecode analysis.
- Deep wallet clustering.
- CEX/router/pool wallet labeling as fact.

### Проверка

- High holder concentration produces a high-risk factor.
- Missing holder snapshot becomes missing data.
- Proxy without implementation metadata is flagged or marked unknown.
- Unverified source affects risk/confidence.

---

## 12. Этап 6 — Liquidity pool features v0

**Статус:** `PLANNED`

### Цель

Вычислить первые liquidity-pool risk factors from demo data.

### Результаты

Feature modules для:

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

### Критерии приёмки

- Pool analysis returns liquidity, APR, and withdrawal-risk factors.
- TVL/APR values include source and timestamp.
- Low liquidity cannot produce high confidence.
- High APR is not treated as positive without sustainability checks.
- Missing LP ownership or APR source is visible.

### Не входит в scope

- Full DEX aggregator.
- Exact execution simulation.
- Production-grade slippage engine.
- Full reward-emission modeling.

### Проверка

- Low TVL raises liquidity risk.
- High APR with low TVL raises sustainability risk.
- Missing APR source becomes unknown or reduces confidence.
- Partial liquidity history lowers confidence.

---

## 13. Этап 7 — Explainable risk scoring v0

**Статус:** `PLANNED`

### Цель

Преобразовать factor-level evidence в transparent risk score и отдельный confidence score.

### Результаты

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

Эти weights являются placeholders и должны быть видимы в output.

### Критерии приёмки

- Score is explainable at factor level.
- Weights are documented.
- Confidence is separate from score.
- Missing data reduces confidence.
- `Insufficient data` is a valid result.
- One metric cannot dominate silently.

### Не входит в scope

- Final calibrated model.
- ML scoring.
- Investment recommendations.

### Проверка

- No factors -> `Insufficient data`.
- Missing critical data lowers confidence.
- Demo fixture returns stable expected result.
- High-risk fixture produces high-risk factors with evidence.
- Score-only output is impossible.

---

## 14. Этап 8 — Генерация Markdown report

**Статус:** `PLANNED`

### Цель

Сгенерировать первые human-readable reports из structured risk results.

### Результаты

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

### Критерии приёмки

- Report includes object identity.
- Report includes sources and timestamps.
- Report includes confidence.
- Report includes missing data and limitations.
- Report can be reused as public demo/case-study material.
- Report contains no investment-advice wording.

### Не входит в scope

- Confidential client reports in public repository.
- Legal/audit conclusions.
- PDF polish before Markdown works.

### Проверка

- Required sections test.
- Wording safety test.
- Source metadata presence test.
- Snapshot test for demo reports.

---

## 15. Этап 9 — CLI/script entry point

**Статус:** `PLANNED`

### Цель

Дать простой локальный способ запускать demo analysis end to end.

### Результаты

- local script or CLI command;
- config-driven input;
- deterministic output path;
- generated demo report.

### Критерии приёмки

- Developer can run demo analysis from one command or script.
- No network is required for the first demo.
- Output is reproducible.
- Errors are explicit.

### Не входит в scope

- Production CLI.
- Live API ingestion.
- Complex command-line interface.

### Проверка

- End-to-end demo run.
- Snapshot report comparison.
- Failure-mode test for missing fixture.

---

## 16. Этап 10 — FastAPI MVP

**Статус:** `PLANNED`

### Цель

Expose stable core logic through a simple local API.

### Результаты

- `GET /health`
- `POST /v1/analyze/token`
- `POST /v1/analyze/pool`
- `POST /v1/report`
- API response schema;
- error schema;
- input validation tests;
- OpenAPI schema.

### Критерии приёмки

- API rejects invalid addresses and unsupported chains.
- Missing `chain_id` returns validation error.
- API returns factors, confidence, sources, missing data, and limitations.
- API failure does not return low risk by default.
- No secrets are logged.

### Не входит в scope

- Authentication.
- Billing.
- Public hosted production API.
- Live data source dependency.

### Проверка

- Healthcheck test.
- Invalid address test.
- Missing `chain_id` test.
- Response shape test.
- Error schema test.
- No-secret logging review.

---

## 17. Этап 11 — Public demo assets and case studies

**Статус:** `PLANNED`

### Цель

Превратить technical MVP в credible portfolio and sales asset.

### Результаты

- 3–5 demo reports;
- example API responses;
- case study pages;
- screenshots or report previews if useful;
- clear disclaimer;
- “What this project demonstrates” section.

### Критерии приёмки

- A potential client can see the deliverable before paying.
- Reports demonstrate liquidity, holders, contract, scoring, confidence, and missing-data logic.
- The repository supports outbound credibility.
- Demo reports do not expose client data or secrets.

### Не входит в scope

- Waiting for inbound clients from publications only.
- Guaranteed scam detection claims.
- Client-specific confidential reports.

### Проверка

- Demo report review.
- Red-team wording review.
- Reproducibility check.

---

## 18. Этап 12 — Коммерческая валидация manual reports

**Статус:** `PLANNED`

### Цель

Проверить, готовы ли users платить за DeFi risk analysis до построения SaaS.

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

Эти prices являются hypotheses и должны быть validated.

### Результаты

- free mini-check offer;
- paid report template;
- outbound message templates;
- client intake form;
- objection log;
- feedback loop into report/scoring methodology.

### Критерии приёмки

- 20–50 targeted outbound messages sent.
- At least 5 free mini-checks completed.
- At least 1 paid report sold or strong buying signal collected.
- Common objections are documented.
- Paid deliverable scope is clear.

### Не входит в scope

- Full SaaS billing.
- Large marketing campaigns.
- Vague “crypto analytics” positioning.

### Проверка

- Outreach metrics.
- Conversion tracking.
- Client feedback review.
- Scope creep review.

---

## 19. Этап 13 — Адаптеры external data sources

**Status:** `DEFERRED` until demo pipeline is stable

### Цель

Добавить live data integrations только после стабилизации contracts, fixtures, scoring, reports и API shape.

### Результаты

- explorer adapter;
- RPC adapter;
- DEX/pool data adapter;
- TVL/APR source adapter;
- timeout/retry/rate-limit logic;
- raw data cache with provenance.

### Критерии приёмки

- One real token or pool can be analyzed with source timestamps and limitations.
- Pagination is handled.
- Partial failures do not become low risk.
- Raw data is cached with provenance.
- Data-source costs and rate limits are understood.

### Не входит в scope

- Broad multi-chain coverage.
- Expensive backfills.
- Production SLA.

### Проверка

- Pagination test.
- Timeout test.
- Partial failure test.
- Source timestamp test.
- Raw cache provenance test.

---

## 20. Этап 14 — Dashboard MVP

**Status:** `DEFERRED`

### Цель

Создать visual interface for reviewing risk results после стабилизации report/API flow.

### Результаты

- risk summary page;
- factor breakdown panel;
- confidence/data completeness panel;
- liquidity panel;
- holder concentration panel;
- contract red flags panel;
- report export button.

### Критерии приёмки

- Dashboard explains why a score is high or low.
- Missing data is visible.
- User can export or open a report.
- Score is never shown without factor evidence.

### Не входит в scope

- Advanced multi-user SaaS.
- Complex permissions.
- Billing.

### Проверка

- UX review.
- Missing-data state review.
- Report export test.

---

## 21. Этап 15 — Monitoring and alerts

**Status:** `DEFERRED`

### Цель

Detect risk deterioration over time for selected tokens, pools, protocols, or portfolios.

### Результаты

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

### Критерии приёмки

- Every alert has object, source, timestamp, severity, threshold, and next action.
- Missing data can trigger warning or manual review.
- Alert deduplication prevents spam.
- API/source failure creates a data-source alert, not a low-risk output.

### Не входит в scope

- Automated trading actions.
- Wallet transaction signing.
- Liquidation or trading recommendations.

### Проверка

- Threshold simulation.
- Deduplication test.
- Missing-data behavior test.
- Alert payload review.

---

## 22. Этап 16 — Hosted API / SaaS evolution

**Status:** `DEFERRED`

### Цель

Перейти от local tooling and manual reports к hosted paid product только после demand validation.

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

Не строить SaaS, пока не выполняется хотя бы несколько условий:

- multiple users request repeated reports;
- manual report workflow becomes repetitive;
- users ask for monitoring;
- users ask for API access;
- at least one paying customer wants recurring checks;
- data-source costs and usage limits are understood.

### Не входит в scope before triggers

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

Остановить implementation и вернуться к design review, если:

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

## 27. Recommended first 30 days

### Week 1 — Foundation

- Finalize README files.
- Add package skeleton.
- Add `.gitignore` and `.env.example`.
- Add `configs/chains.yaml`.
- Add initial test setup.

### Week 2 — Contracts and demo fixtures

- Implement core schemas.
- Implement chain/address validation.
- Add demo token fixture.
- Add demo pool fixture.
- Add fixture loading tests.

### Week 3 — Features and scoring

- Implement token feature extraction v0.
- Implement liquidity feature extraction v0.
- Implement scoring v0.
- Implement confidence v0.
- Add snapshot tests.

### Week 4 — Reports and first public demos

- Implement Markdown report generator.
- Generate token demo report.
- Generate pool demo report.
- Add example API response shape.
- Prepare first case-study style README section.

---

## 28. Roadmap update policy

Обновлять этот roadmap, когда:

- a phase changes status;
- a schema changes;
- scoring logic changes;
- confidence logic changes;
- API response shape changes;
- report structure changes;
- external data adapters are introduced;
- a commercial assumption is validated or disproven;
- a deferred subsystem becomes active.

Когда implementation начинается, каждая phase должна быть помечена одним из статусов:

- `PLANNED`
- `IN_PROGRESS`
- `IMPLEMENTED`
- `PARTIAL`
- `BLOCKED`
- `DEFERRED`
- `REMOVED`

Do not let this roadmap become a marketing document. Marketing positioning belongs in `README_marketing.md`. Public project explanation belongs in `README.md`. Implementation details belong in `README_implementation_blueprint.md`.
