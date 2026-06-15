# DeFi Risk Lens

**DeFi Risk Lens** — это проект для объяснимого анализа рисков DeFi и on-chain-активов: токенов, пулов ликвидности, протоколов и криптопортфелей.

Название репозитория:

```text
defi-risk-analyzer
```

Цель проекта — не прогнозировать цены, не генерировать торговые сигналы и не обещать прибыль.

Цель проекта — сделать риски DeFi более видимыми, структурированными и основанными на проверяемых данных.

## Что анализирует этот проект

DeFi Risk Lens фокусируется на таких факторах риска, как:

- глубина ликвидности и изменения ликвидности;
- концентрация держателей;
- активность китов;
- подозрительные переводы;
- тревожные признаки в смарт-контрактах;
- риски прокси-контрактов и административного контроля;
- логика mint, burn, blacklist, pause и ограничений на переводы;
- устойчивость TVL и APR;
- волатильность и рыночный риск;
- зависимости протоколов;
- недостающие данные и ограничения уверенности анализа.

## Основной принцип

Risk score никогда не должен быть «магическим числом».

Каждый результат анализа риска должен объяснять:

- какие признаки были использованы;
- что повысило или снизило риск;
- какие источники данных были использованы;
- каких данных не хватает;
- насколько уверенным является анализ;
- что нужно проверить вручную на следующем этапе.

## Features

Запланированные и/или находящиеся в разработке возможности:

- анализ риска токена;
- анализ риска пула ликвидности;
- краткий анализ риска протокола;
- анализ концентрации держателей;
- сводка активности китов;
- чек-лист тревожных признаков смарт-контракта;
- объяснимый risk scoring;
- отчётность по confidence и missing data;
- структура risk report в стиле HTML/PDF;
- формат ответа, готовый для API;
- будущие правила мониторинга и alerting.

Более подробная документация по возможностям будет добавлена позже.

## Intended output

Проект предназначен для формирования структурированных результатов анализа риска, таких как:

- token risk reviews;
- liquidity pool risk reviews;
- protocol risk summaries;
- holder concentration analysis;
- whale activity summaries;
- smart-contract red-flag checks;
- HTML/PDF risk reports;
- API-ready risk responses;
- future monitoring and alerting rules.

## Risk levels

Предполагаемые уровни риска:

- `Low risk`
- `Medium risk`
- `High risk`
- `Critical risk`
- `Insufficient data`

Если доступные данные неполные, устаревшие или ненадёжные, корректный результат — не уверенный score. Корректный результат — сниженная confidence или `Insufficient data`.

## Example report

Примеры отчётов будут добавлены позже.

Планируемая структура отчёта:

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

Примеры API будут добавлены позже.

Планируемая структура ответа:

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

### Phase 1 — Основа проекта

- Определить структуру проекта.
- Определить формат risk report.
- Определить принципы risk scoring.
- Добавить demo data.
- Добавить первые статические example reports.

### Phase 2 — Анализ токенов и пулов

- Реализовать извлечение metadata токена.
- Добавить проверки концентрации держателей.
- Добавить метрики пула ликвидности.
- Добавить базовые проверки red flags смарт-контракта.
- Добавить логику missing data и confidence.

### Phase 3 — Explainable risk scoring

- Определить веса признаков.
- Добавить factor-level contributions.
- Добавить confidence score.
- Добавить validation examples.
- Добавить sanity checks на известных кейсах.

### Phase 4 — Reports and API

- Генерировать структурированные отчёты в стиле HTML/PDF.
- Добавить схему API response.
- Добавить простой FastAPI endpoint.
- Добавить examples requests and responses.

### Phase 5 — Monitoring and commercial layer

- Добавить alerting rules.
- Добавить мониторинг ликвидности и китов.
- Добавить концепцию уведомлений через Telegram/email.
- Добавить workflow платного risk report.
- Добавить roadmap для hosted API/SaaS.

## What this project is not

Этот проект **не является**:

- финансовой рекомендацией;
- торговым ботом;
- инструментом прогнозирования цены;
- гарантией того, что токен, пул или протокол безопасен;
- инструментом для автоматической покупки, продажи или подписания транзакций;
- полноценным формальным аудитом смарт-контракта.

Проект может выявлять индикаторы риска, но он не гарантирует, что актив является безопасным или небезопасным.

## Safety and data handling

Этот проект никогда не должен запрашивать или хранить:

- seed phrases;
- private keys;
- wallet passwords;
- exchange passwords;
- hardcoded API secrets.

Весь анализ должен основываться на публичных on-chain-данных, публичных данных протоколов, предоставленных пользователем нечувствительных данных или безопасных API-интеграциях.

## Commercial direction

DeFi Risk Lens задуман одновременно как:

- открытый технический portfolio project;
- основа для платных DeFi risk reports, мониторинга и API-based risk analysis.

Публичный репозиторий демонстрирует методологию и инженерный подход.

Коммерческая ценность может формироваться за счёт:

- свежих данных;
- hosted infrastructure;
- monitoring;
- custom reports;
- integrations;
- support;
- expert interpretation.

## License

Этот проект распространяется по лицензии **Apache License 2.0**.

Подробности см. в файле `LICENSE`.

## Disclaimer

Этот проект предназначен только для образовательных, аналитических и risk-research целей.

Он не предоставляет инвестиционные советы, финансовые рекомендации, торговые сигналы или гарантии безопасности либо прибыльности.

Всегда проводите самостоятельную проверку перед взаимодействием с любым DeFi-активом, пулом, протоколом или смарт-контрактом.