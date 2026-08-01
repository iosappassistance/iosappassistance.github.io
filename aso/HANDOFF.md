# Передача контекста другому Claude

## 1. Секреты и переменные

MCP-сервер App Store Connect читает ровно три переменные окружения:

| Переменная | Значение |
|---|---|
| `ASC_KEY_ID` | `K4V2R5MNW8` |
| `ASC_ISSUER_ID` | `cbb47998-d4db-48f6-bd54-7a4d07d9d645` |
| `ASC_PRIVATE_KEY_PATH` | `~/.appstoreconnect/private_keys/AuthKey_K4V2R5MNW8.p8` |

Всё это уже прописано в `~/.claude.json` → `mcpServers.appstore-connect.env`.

**Приватный ключ `.p8` не хранится в этом репозитории и не должен в него попадать.**
Он скачивается из App Store Connect ровно один раз. Если нужно перенести на другую машину —
копировать файл вручную по защищённому каналу, положить в `~/.appstoreconnect/private_keys/`
и поставить `chmod 600`. Через чат ключ не передавать.

Роль ключа в ASC: App Manager (хватает на метаданные и скриншоты).

## 2. Что ещё может понадобиться

| Что | Значение |
|---|---|
| AdMob Publisher ID | `pub-5815150517117838` |
| AdMob App ID (DriveGE) | `ca-app-pub-5815150517117838~1827148217` |
| Домен | `appassistance.services` (GoDaddy → GitHub Pages) |
| Репозиторий сайта | `dumojo/appassistance` (ветка `main`, корень) |
| Контактная почта | `iosappassistance@gmail.com` |
| Проект приложения | `~/Downloads/DriveGE/DriveGE.xcodeproj`, схема `DriveGE`, bundle `com.ieAD.DriveGE` |

Ключ `AuthKey_4GMMLV8N74.p8` в той же папке — **нерабочий для ASC API** (даёт 401,
скорее всего от RevenueCat или APNs). Не путать.

## 3. Идентификаторы App Store Connect

Машиночитаемо — в `asc-ids.json` рядом. Кратко:

- App ID: `6774959850`
- Версия 1.2.0: `c58e5869-06b3-4952-ac08-aec508822e07` (состояние `PREPARE_FOR_SUBMISSION`)
- Локализации версии: `ru` → `28469349-…`, `en-US` → `8518c491-…`
- Наборы скриншотов `APP_IPHONE_65`: ru → `102693a1-…`, en-US → `1f0fcce3-…`

## 4. Состояние работы

**Сделано:**
- Сайт `appassistance.services` с 7 подстраницами, app-ads.txt, SEO, llms.txt, HTTPS
- Terms для всех приложений на сайте, privacy для DriveGE
- Метаданные 1.2.0 залиты (имя, подзаголовок, ключевики, описание с дисклеймером, promo text, privacy URL)
- 12 скриншотов залиты в 1.2.0: грузинские в слот `en-US`, русские в `ru`, статус `COMPLETE`

**Не сделано / за пользователем:**
- Release notes в 1.2.0 — сейчас шаблон, заменить
- Загрузить сборку, отправить на ревью
- Google Search Console + sitemap
- AdMob: аккаунт отклонён, ждать одобрения (реклама не идёт, ~3 DAU)
- Экология (эко-менеджмент) в базе вопросов отсутствует — отдельная часть грузинского экзамена, ни у кого из конкурентов нет на русском

## 5. Что потеряно и как восстановить

Рабочая папка сессии (`/private/tmp/claude-501/.../scratchpad`) — временная и уже подчищена.
Удалены генераторы (`make_series.py`, `ms_fix.py`, `build_all.py`, `upload_shots.py`),
исходные снимки экранов приложения и финальные `ka_*.png` / `ru_*.png`.

Уцелело и лежит в `aso/screenshots/`: английский комплект `en_1..6.png` и контактные листы.

**12 залитых скриншотов не потеряны** — они на серверах Apple и скачиваются через API:
`GET /v1/appScreenshotSets/{id}/appScreenshots`, поле `imageAsset.templateUrl`
(подставить ширину, высоту и формат вместо `{w}`, `{h}`, `{f}`).

Исходные снимки экранов пересобираются: собрать приложение в симуляторе, засеять данные
напрямую в SwiftData-хранилище `default.store` (массивы `questionIds`/`answers` в `TestResult`
хранятся как **NSKeyedArchiver**, не JSON и не обычный plist — иначе приложение падает на старте),
переключать язык через `defaults write com.ieAD.DriveGE appLanguage ka|ru|en`.
