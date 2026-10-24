---
type: component
created: 2026-10-22
updated: '2026-10-24'
status: planned
tags:
- drive2u
- component
source: 'drive2u/docs/app-requirements.md — FR-01–FR-10; розділи 5–8; drive2u@7147aba786978b2aec6411294d5837313d2b7dde:
  server/Drive2U.Api/FareEstimatesController.cs, Program.cs; server/Drive2U.Fares/FareCalculator.cs,
  FareTariff.cs; server/README.md; docs/orchestration/backend-fare/verification.md'
---

# Серверний API

## Відповідальність
ASP.NET Core Web API — джерело істини для прав, вартості й стану [[Ride]]. Використовує Identity, EF Core та початково SQLite. Запланована директорія: server/. Компонент описує цільову архітектуру, не підтверджує наявність коду.

## Контракти
Приймає запити автентифікації, профілю, [[Fare-Quote]], створення/прийняття поїздки, переходів і історії. Формат маршрутів API та DTO буде погоджено окремо. Неавтентифіковані запити — 401; конкурентні конфлікти — 409; валідаційні та ресурсні помилки мають стабільний контракт.

## Бізнес-логіка
Перевіряє ролі, власника, доступність [[Driver]], чинність пропозиції, обмеження активних поїздок та [[Ride-Lifecycle]]. Конкурентне прийняття й скасування виконуються транзакційно. Публікація [[Realtime-Updates]] відбувається після збереження; повторне читання API усуває пропущені клієнтом події.

## Інтеграції та перевірки
Від [[Mapping-Integration]] отримує достовірну відстань. Обробляє тайм-аути без підміни даних. xUnit перевіряє доменну логіку; інтеграційні тести — фактичну SQLite-конкурентність, авторизацію та ідемпотентність. Підстава: [[ADR-002-Server-Authority]].

## Підтверджений частковий етап — 2026-10-24
На ревізії Drive2U `7147aba786978b2aec6411294d5837313d2b7dde` реалізовано початковий ASP.NET Core API у `server/` та незалежний модуль `Drive2U.Fares`. Попередні розділи описують цільову архітектуру; Identity, EF Core/SQLite, авторизація, серверна маршрутизація, повний Fare-Quote, замовлення/їх життєвий цикл, SignalR і mobile ще не реалізовані. Статус усього компонента залишається planned.

`GET /health` повертає 200 із `{"status":"ok"}`. `POST /api/fare-estimates` приймає лише числове `distanceKm > 0` і повертає distanceKm, amount та currency=UAH. Калькулятор використовує decimal, формулу `BaseFare + DistanceKm * RatePerKm`, checked arithmetic та єдине округлення кінцевої суми до двох знаків `MidpointRounding.AwayFromZero`. Тариф задається серверною конфігурацією; BaseFare і RatePerKm обов’язкові та невід’ємні, некоректна конфігурація зупиняє запуск. Приклад 40/12 — конфігураційний приклад, не затверджений ринковий тариф.

Відсутній/null/некоректний body, нечислове, нульове, від’ємне або завелике distanceKm, arithmetic overflow і невідомі поля (зокрема клієнтський тариф, currency та amount) відхиляються 400 problem JSON без деталей винятків. Клієнт не змінює серверний тариф.

Це демонстраційний estimate за недовіреною клієнтською відстанню, а не [[Fare-Quote]]: немає quoteId, строку дії, серверної дорожньої відстані чи фіксації ціни замовлення. Повний FR-03 та весь Backend-API не завершені.

Історичний результат на цій ревізії, за `docs/orchestration/backend-fare/verification.md`: Coder і незалежний Reviewer підтвердили build без warnings/errors та 37 passed, 0 failed/skipped (10 business, 20 HTTP, 7 startup cases). Loopback smoke Coder: health успішний; 2.5 км за прикладом 40/12 → 70 UAH; створений процес завершено. У поточній синхронізації ці тести повторно не запускалися; код server/ відтоді не змінено.
