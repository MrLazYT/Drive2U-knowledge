---
type: component
created: 2026-10-24
updated: 2026-10-24
status: active
tags: [drive2u, component]
source: https://github.com/MrLazYT/Drive2U/commit/236eea02af6c70df0240091503c7e02f79a4ba78;
  drive2u/docs/ontology.md
---

# Додавання MCP-конектора бази знань

## Подія
Повний SHA: 236eea02af6c70df0240091503c7e02f79a4ba78
Повідомлення: added knowledge_mcp
Дата коміту (UTC): 2026-10-23T13:42:04Z.

## Фактичні зміни та відповідальність
Додано 13 файлів у harness/knowledge-mcp; статистика GitHub — 1033 додані рядки. knowledge.py містить операції локального vault: читання, текстовий пошук, валідацію шляхів і метаданих, перевірку ревізій, ingestion та git-синхронізацію. server.py надає STDIO MCP-інструменти health_check, get_note, search_notes та ingest_note.

## Входи, виходи й залежності
Конфігурація використовує корені окремих репозиторіїв застосунку та vault і прапорець дозволу запису. Ingestion приймає заголовок, тип, Markdown, джерело та пов’язані нотатки; повертає статус, ревізію, commit і published. Додано файли Python-залежностей і приклад конфігурації Codex.

## Основні файли
- harness/knowledge-mcp/knowledge.py
- harness/knowledge-mcp/server.py
- harness/knowledge-mcp/examples/codex-config.toml
- harness/knowledge-mcp/requirements.txt, requirements-dev.txt, requirements-lock.txt
- harness/knowledge-mcp/README.md та README (1).md
- harness/knowledge-mcp/VERIFICATION.md
- harness/knowledge-mcp/tests/test_knowledge.py та tests/test_mcp_transport.py

Також додано два бінарні файли __pycache__/knowledge.cpython-313.pyc та __pycache__/server.cpython-313.pyc.

## Помилки та перевірки
У коді є відмова при вимкненому записі, некоректних шляхах, конфлікті ревізій, зайнятому writer lock та нечистому vault. Невдалий push повертає published=false і зберігає маркер для повтору. У коміті додано тести та документ із результатами перевірок; у межах підготовки цієї нотатки тести не запускалися, їх успішність незалежно не перевірялася.

## Значення для Drive2U
Коміт додає код і документацію контрольованого MCP-доступу до окремої бази знань. Сам факт коміту не підтверджує розгортання конектора в конкретному середовищі або реалізацію функцій мобільного застосунку. Нотатка фіксує зміну репозиторію, а не архітектурне рішення.

## Related notes
- [[Conventions]]
- [[Knowledge-Connector]]

## Typed relations
- governed_by: [[Conventions]]
