---
type: component
created: 2026-10-23
updated: '2026-10-26'
status: active
tags:
- drive2u
- component
source: 'drive2u/harness/knowledge-mcp/README.md; підтверджені MCP-виклики версії
  1.2.0; drive2u/docs/ontology.md; drive2u@5884f32ca4c8d2032aad403c7335dd9cc4ef5f14:
  harness/knowledge-mcp/knowledge.py, server.py, tests; docs/skills-verification.md
  R1/K1; MCP health_check 1.3.0 2026-10-24; drive2u@be24e88ef05239f5be6d07959184a5d28d40b6a9:
  harness/knowledge-mcp/semantic.py, server.py, tests; docs/retrieval/design.md, comparison.md,
  verification.md'
---

# Конектор бази знань

## Призначення
Локальний MCP-сервер надає контрольований доступ до окремого Obsidian vault Drive2U. Працює як STDIO-процес, який запускає Codex.

## Інструменти
- health_check — перевіряє доступність, версію, режим запису та зайнятість writer.
- get_note — читає нотатку та повертає її метадані й ревізію.
- search_notes — виконує локальний семантичний пошук фрагментів дозволених нотаток; явний діагностичний режим text збережено без прихованого fallback.
- ingest_note — створює або оновлює нотатку, додає посилання до MOC, виконує commit і push.

## Підтверджений стан
Версія 1.3.0 підтверджена health_check через Codex 2026-10-24: write_enabled=true, writer_busy=false, pending_push=false. Перевірки стану, читання та пошуку успішні. Публікація попередніх ingestion через Codex задокументована в harness/knowledge-mcp/VERIFICATION.md і docs/guardrails-test-cases.md; результат поточного K1 та точного повтору фіксується окремо в docs/skills-verification.md.

Код на ревізії Drive2U 5884f32ca4c8d2032aad403c7335dd9cc4ef5f14 дозволяє оновити наявний кореневий Conventions.md лише з title="Conventions", note_type="convention". Conventions прочитана через MCP; секція типізованих зв’язків уже наявна, додаткового запису до неї не потрібно. Інші кореневі нотатки через ingestion не записуються.

Перевірка R1 на цій ревізії: 31 passed, 3 skipped (Windows symlink і два POSIX process-group тести), включно зі справжнім MCP/STDIO на одноразових локальних app/vault/bare remote. Це не перевірка Windows taskkill. Реальний vault не використовувався тестами.

## Обмеження
Читання не запускає git і не очікує writer lock. Запис потребує чистого робочого дерева та налаштованого доступу до origin. Зміна існуючої нотатки потребує expected_revision. Помилка push не означає відсутності локального коміту. health_check не перевіряє clean tree або актуальність origin; читання атомарне для окремого файла, не для всього vault. Семантика типізованих зв’язків і цикли перевіряються вручну.

Історично R1 виявив INVALID_YAML: parse_note() обрізав метадані на `---` усередині source. Виправлення 1.3.1 та відновлення Backend-API підтверджені drive2u@d13931d4b87eae022ae54898e90c218c70f5807e і реальними MCP readback; поточне читання Backend-API успішне, це більше не чинна помилка.

## Зв’язки
Конектор підтримує інженерну роботу над Backend API, але не входить до бізнес-логіки перевезень. Формат нотаток визначають конвенції vault. Підключення офіційного GitHub MCP перевіряється окремо.

## Related notes
- [[Backend-API]]
- [[Conventions]]

## Typed relations
- governed_by: [[Conventions]]

## Семантичний retrieval 1.4.0 — підтверджено 2026-10-26
Джерело реалізації: drive2u@be24e88ef05239f5be6d07959184a5d28d40b6a9, harness/knowledge-mcp/{server.py,knowledge.py,semantic.py,tests}; docs/retrieval/{design.md,verification.md,comparison.md}. Активний health_check повернув 1.4.0, write_enabled=true, writer_busy=false, pending_push=false.

Локальна CPU-модель intfloat/multilingual-e5-small зафіксована на revision 614241f622f53c4eeff9890bdc4f31cfecc418b3, 384 виміри; query:/passage: префікси, нормалізовані вектори. SQLite зберігає derived індекс; ранжування — точний dot product/cosine по всіх векторах, без ANN чи зовнішнього сервісу. Offline local safetensors, trust_remote_code=False. Markdown лишається джерелом істини.

Heading chunks враховують ієрархію ATX (без заголовків усередині fenced code), бюджет 1000 символів із контекстом і 512 токенів фактичного tokenizer. Повертаються note_id, title, section, text/excerpt, source, SHA256 revision, chunk_id та cosine score; score не є ймовірністю правильності. total рахує фрагменти, тому можливі повтори однієї нотатки й малокорисні heading-only результати.

Кожен пошук перевіряє актуальні дозволені файли: змінені/нові нотатки переіндексовуються інкрементально, видалені прибираються транзакційно. Повертаються note_revisions, corpus_revision та index_updates. Повторний snapshot відхиляє конкурентні зміни; це не транзакційний snapshot vault. Помилки busy, моделі, YAML, неповного/пошкодженого/несумісного індексу явні, без застарілих результатів чи прихованого text fallback. Derived індекс містить текст і потребує конфіденційності vault; агент читає знання лише через MCP.

Історичні перевірки фінального коду: 64 passed, 5 skipped за 86.00 s; окремий opt-in real-model STDIO — 1 passed за 22.62 s. Windows startup завантажує залежності Sentence Transformers до STDIO reader threads, без побудови моделі/індексу; це усуває відтворене зависання native NumPy/SciPy import, але додає вартість запуску. Старі вже запущені процеси потребують заміни. У цьому продовженні Windows Restart Manager встановив єдиного власника index.lock: застарілий MCP PID 2936, запуск 15:10:50 до зміни server.py о 15:17:54 (+02:00). Після завершення лише цього процесу semantic MCP search успішний; новий MCP PID 4284 запущений о 15:41:43 після виправлення. Lock-файли не видалялися.

Фіксоване порівняння 10 питань на незмінних 19 нотатках показало змішаний результат: середня релевантність 40% → 40%, покриття очікуваних джерел 51.67% → 36.67% (погіршення 15 процентних пунктів), повнота перевірених фактів відповіді 65% → 61.67% (погіршення 3.33 пункту). Середній час MCP search 56.7 → 261.2 ms. Q4 покращився; загальна перевага semantic не доведена. Baseline, питання, критерії та оцінки не змінено. Ця синхронізація відбувається після порівняння і змінює наступну ревізію корпусу, не його зафіксовані результати. Демонстраційний fare estimate не є готовим Fare-Quote.
