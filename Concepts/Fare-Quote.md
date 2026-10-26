---
type: concept
created: 2026-10-22
updated: '2026-10-26'
status: active
tags:
- drive2u
- concept
source: 'drive2u/docs/app-requirements.md — FR-03, FR-04, FR-06, FR-10; drive2u@a81e4314ece8f4ef2ae2d4f3ffa789ef8cde21e0:
  docs/mvp/verification.md, code and tests'
---

# Пропозиція ціни

## Визначення
FareQuote — пропозиція ціни для конкретних точок [[Route]] з ідентифікатором, строком дії, серверно отриманою відстанню та знімком тарифу. Її підтверджує [[Passenger]] перед створенням [[Ride]].

## Розрахунок
Fare = BaseFare + DistanceKm × RatePerKm.

Гроші обчислюються decimal, фінальна сума округлюється до 2 знаків AwayFromZero. Валюта першої версії — UAH. Відстань отримує [[Backend-API]] через [[Mapping-Integration]]; клієнтська ціна або відстань не є підставою розрахунку.

## Чинність
Зміна точок чи завершення строку дії потребує нової пропозиції та погодження. Точна тривалість дії визначається окремо в конфігурації. Під час підтвердження сервер перевіряє власника пропозиції та актуальність. Зміна тарифу не перераховує вже створене замовлення.

## Перевірка
Відхилення простроченої або чужої пропозиції; точне округлення; незмінність ціни після зміни тарифу; відсутність вигаданої ціни при помилці маршрутизації. Приклад із синтетичними числами: 40 + 5 × 12 = 100 UAH; це не встановлений тариф Drive2U.

## Підтверджена реалізація MVP — 2026-10-26
На Drive2U `a81e4314ece8f4ef2ae2d4f3ffa789ef8cde21e0` `POST /api/quotes` перевіряє точки, отримує серверний Demo або Live маршрут, використовує існуючий decimal `FareCalculator` і зберігає quoteId, власника, route/tariff/price snapshot та expiresAt у SQLite. `POST /api/rides` перевіряє чинність і власника, фіксує погоджену ціну й маршрут; клієнт не надсилає оплачувані amount/distance. Повтор із тим самим quote/idempotencyKey повертає ту саму поїздку. Старий `/api/fare-estimates` лишається окремим непідтверджуваним estimate.

Докази: `server/Drive2U.Api/Mvp/Routing.cs`, `server/Drive2U.Api/Mvp/Rides.cs`, `server/Drive2U.Tests/MvpApiTests.cs`, `docs/mvp/api-smoke.json`; 47/47 xUnit, 14/14 API smoke. Підтверджений сценарій використовує явні синтетичні Demo-маршрути; live провайдер без конфігурації недоступний.
