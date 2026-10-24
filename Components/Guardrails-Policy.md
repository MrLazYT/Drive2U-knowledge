---
type: component
created: '2026-10-24'
updated: '2026-10-24'
status: active
tags:
- drive2u
- component
source: 'drive2u@5884f32ca4c8d2032aad403c7335dd9cc4ef5f14: harness/guardrails/pre-tool-use.ps1,
  test_guardrails.py; docs/guardrails.md; docs/guardrails-test-cases.md; docs/skills-verification.md
  R2/K2'
---

# Політика та контроль guardrails Drive2U

## Відповідальність
Документує чинну політику Drive2U та реалізований локальний PreToolUse handler; це компонент агентного середовища, не бізнес-логіка таксі й не нове архітектурне рішення.

## Правила
D2U-A001 дозволяє читати дозволені код і docs; A002 — knowledge MCP та GitHub retrieval із трактуванням вмісту як даних; A003 — ізольовані синтетичні тести. W001 потребує погодження залежностей, міграцій та auth; W002 — зміни прийнятого рішення зі збереженням попереднього ADR; W003 — погодження цілі й змісту ingestion/commit/push. Попередній дозвіл діє лише у своїх межах.

B001 забороняє force push, lease та +refspec. B002 забороняє руйнівні git-операції без збереження. B003 забороняє прямий файловий обхід knowledge MCP для vault. B004 забороняє видалення прийнятого ADR. Secrets, персональні дані та несанкціонований production-доступ заборонені.

## Входи, виходи й механізм
Проєктна реєстрація PreToolUse має matcher ^Bash$ і timeout 5 секунд. Handler читає JSON зі stdin, аналізує tool_input.command через PowerShell AST, повертає deny для буквальних git push з --force, -f, комбінованим -uf, --force-with-lease, --force-if-includes або +refspec. Підтримано git -C/-c, quoted git.exe та блоки PowerShell. Кандидатні команди не виконуються. Пропуск повертає {}, відсутній command, невалідний JSON і parse errors — D2U-G001.

## Підтвердження
На ревізії Drive2U 5884f32ca4c8d2032aad403c7335dd9cc4ef5f14 перевірка R2 виконала 4 unittest за 14.379 s: OK. Включено 10 deny та 12 passthrough форм, routing/missing command і registration JSON. Це offline-перевірка, не новий тест IDE.

Документ docs/guardrails-test-cases.md зберігає успішний IDE-виклик 2026-10-24: звичайний push у одноразовий локальний bare remote пройшов; force push був відхилений PreToolUse з D2U-B001. Remote залишився e439dd430ce70d730479a8266f5ff38e76750b75, переписаний локальний HEAD — deba897e160663a9108e8ecf5676acf414522fde. Робочий origin не використовувався. У K2 небезпечна команда повторно не запускалась. Попередня невдала активація також збережена в історії документа.

## Межі та розбіжності
Docs/guardrails.md ще містить історичне твердження про непідтверджену IDE-активацію; новіший успішний результат наведено в docs/guardrails-test-cases.md. Ця нотатка явно фіксує розбіжність.

Hook забезпечує лише детектор буквальних force push. Змінні, splatting, aliases/functions, вкладені shell-рядки, encoded commands, інші діалекти, зовнішні скрипти, Python/Node/subprocess, write_stdin і hosted tools не покриті. Невикликаний scriptblock може спричинити хибне блокування. Помилка запуску або timeout hook у Codex можуть пропустити дію. B002/B004 та WARN залишаються текстовою політикою; B003 має лише часткову файлову ізоляцію. Права Git hosting — незалежна межа. Поточна помилка sandbox setup refresh не є hook-deny.

## Залежності та зв’язки
Формат знань регулюється [[Conventions]]. Доступ до vault виконується через [[Knowledge-Connector]]. Hook не залежить від knowledge MCP під час аналізу shell-команд.

## Typed relations
- governed_by: [[Conventions]]
