# ZAFLA Audits Database

**Автор и правообладатель: Davyd Kochuhur (Zero Azimuth Vivifactor)**
ZAFLA — Zero Azimuth Full Liability Authority · Протокол `BiCA a8f3c9d2e1b40571`

Единая самодостаточная база данных **1026 документов аудита** (647 DOCX, 374 PDF, 5 TXT).
Каждый документ хранится в трёх видах: оригинальные байты (BLOB), дословный извлечённый
текст и читаемая копия, плюс метаданные и полнотекстовый FTS5-индекс для ИИ и поиска.

- 📦 База: **`AUDITS_DB.sqlite`** (~1.28 ГБ) — скачивается из [Releases](https://github.com/Daothecreator/17/releases)
- 🔎 Полнотекстовый поиск по всем документам (SQLite FTS5)
- 🤖 Читается любым ИИ/скриптом: обычный SQLite, без внешних зависимостей
- ✍️ Авторство и идентификаторы автора (`ZAFLA-EXTRACT-RU-2026-0622-Ω`, `ZAFLA-MASTER-2026-01` и др., 19 шт.) в таблицах `authors`, `signatures`, `certifications`
- ✅ Целостность: SHA-256 каждого файла и всей базы в `manifest.json` / `checksums.txt`

## Быстрый старт

```bash
git clone https://github.com/Daothecreator/17.git
cd 17
python src/download_db.py        # скачает AUDITS_DB.sqlite из релиза и проверит SHA-256
python examples/quickstart.py    # демо: статистика, поиск, извлечение оригинала
```

Зависимостей нет — только Python 3.9+ (стандартная библиотека).

## Подключение как API / база для приложений

```bash
python src/server.py --host 0.0.0.0 --port 8017
```

Откройте `http://localhost:8017` — веб-поиск; JSON API:

| Эндпоинт | Описание |
|---|---|
| `GET /api/search?q=BlackRock&limit=20` | полнотекстовый поиск со сниппетами |
| `GET /api/files?ext=.pdf` | список документов |
| `GET /api/file/{id}?text=1` | метаданные + текст документа |
| `GET /api/file/{id}/original` | скачать оригинальный файл |
| `GET /api/stats`, `/api/meta` | статистика и метаданные |
| `GET /api/signatures`, `/api/certifications` | подписи и идентификаторы автора |

## Использование из Python

```python
from src.audits_db import AuditsDB

with AuditsDB() as db:
    print(db.stats())
    for hit in db.search("сертификация", limit=5):
        print(hit["name"], "—", hit["snippet"])
    db.save_original(1, out_dir="out")   # извлечь оригинальный файл из BLOB
```

Или напрямую любым SQLite-клиентом (DB Browser, datasette, sql.js для веб-публикации):

```sql
SELECT name FROM files_fts JOIN files f ON f.id = files_fts.rowid
WHERE files_fts MATCH 'Kochuhur';
```

## Структура репозитория

| Путь | Назначение |
|---|---|
| `src/audits_db.py` | клиентская библиотека (поиск, документы, оригиналы, подписи) |
| `src/server.py` | HTTP JSON API + веб-страница поиска (stdlib) |
| `src/download_db.py` | загрузка базы из релиза с проверкой SHA-256 |
| `examples/quickstart.py` | примеры использования |
| `docs/SCHEMA.md` | полная документация схемы БД |
| `manifest.json` | реестр всех 1026 файлов с SHA-256 + хеш базы |
| `checksums.txt` | контрольные суммы релизных файлов |

## Публикация и верификация

База распространяется через GitHub Releases (файл `AUDITS_DB.sqlite`, ~1.28 ГБ).
После скачивания проверьте подлинность:

```
SHA-256 = 4570eaec8c720988d82532cc2ec05bf033bd73274e6ef4233a0e8fa5822b67e8
```

`python src/download_db.py` делает это автоматически. Для публикации как веб-датасета
подойдут [datasette](https://datasette.io/) или [sql.js](https://sql.js.org/) + GitHub Pages.

## Лицензия и атрибуция

MIT (см. `LICENSE`), © 2026 Davyd Kochuhur (Zero Azimuth Vivifactor).
При использовании и распространении указывайте авторство.
