---
type: component
created: 2026-10-23
updated: 2026-10-24
status: active
tags: [drive2u, component]
source: 'drive2u/harness/knowledge-mcp/README.md; підтверджені MCP-виклики версії
  1.2.0; drive2u/docs/ontology.md; drive2u@5884f32ca4c8d2032aad403c7335dd9cc4ef5f14:
  harness/knowledge-mcp/knowledge.py, server.py, tests; docs/skills-verification.md
  R1/K1; MCP health_check 1.3.0 2026-10-24'
---

# Конектор бази знань

## Призначення
Локальний MCP-сервер надає контрольований доступ до окремого Obsidian vault Drive2U. Працює як STDIO-процес, який запускає Codex.

## Інструменти
- health_check — перевіряє доступність, версію, режим запису та зайнятість writer.
- get_note — читає нотатку та повертає її метадані й ревізію.
- search_notes — виконує текстовий пошук серед дозволених нотаток.
- ingest_note — створює або оновлює нотатку, додає посилання до MOC, виконує commit і push.

## Підтверджений стан
Версія 1.3.0 підтверджена health_check через Codex 2026-10-24: write_enabled=true, writer_busy=false, pending_push=false. Перевірки стану, читання та пошуку успішні. Публікація попередніх ingestion через Codex задокументована в harness/knowledge-mcp/VERIFICATION.md і docs/guardrails-test-cases.md; результат поточного K1 та точного повтору фіксується окремо в docs/skills-verification.md.

Код на ревізії Drive2U 5884f32ca4c8d2032aad403c7335dd9cc4ef5f14 дозволяє оновити наявний кореневий Conventions.md лише з title="Conventions", note_type="convention". Conventions прочитана через MCP; секція типізованих зв’язків уже наявна, додаткового запису до неї не потрібно. Інші кореневі нотатки через ingestion не записуються.

Перевірка R1 на цій ревізії: 31 passed, 3 skipped (Windows symlink і два POSIX process-group тести), включно зі справжнім MCP/STDIO на одноразових локальних app/vault/bare remote. Це не перевірка Windows taskkill. Реальний vault не використовувався тестами.

## Обмеження
Читання не запускає git і не очікує writer lock. Запис потребує чистого робочого дерева та налаштованого доступу до origin. Зміна існуючої нотатки потребує expected_revision. Помилка push не означає відсутності локального коміту. health_check не перевіряє clean tree або актуальність origin; читання атомарне для окремого файла, не для всього vault. Семантика типізованих зв’язків і цикли перевіряються вручну.

Відомий INVALID_YAML у Backend-API залишається: існування підтверджене search_notes, вміст не перезаписувався. R1 виявив дефект parse_note(): послідовність `---` усередині YAML-значення source обрізає метадані; виправлення не входило до рев’ю.

## Зв’язки
Конектор підтримує інженерну роботу над Backend API, але не входить до бізнес-логіки перевезень. Формат нотаток визначають конвенції vault. Підключення офіційного GitHub MCP перевіряється окремо.

## Related notes
- [[Backend-API]]
- [[Conventions]]

## Typed relations
- governed_by: [[Conventions]]
