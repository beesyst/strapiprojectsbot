# StrapiProjectsBot

**StrapiProjectsBot** — модульная платформа для сбора, агрегации и структурирования данных о проектах с поддержкой парсинга сайтов, X (Twitter), коллекционных сервисов типа linktr.ee и автоматической очистки данных. Позволяет централизованно управлять настройками, генерировать отчеты в различных форматах и масштабировать систему под любые задачи мониторинга, анализа и агрегации проектов.

## Основные возможности

- **Модульная архитектура** — плагины для сайтов, X-профилей, коллекционных сервисов.
- **Гибкая обработка ссылок** — парсинг из био, коллекционных страниц, генерация чистых ссылок (YouTube, docs, github).
- **Автоматическая нормализация** — удаление хвостов (`/featured`, `/videos`, `/tree/master` и т.д.).
- **Централизованная настройка** — все параметры в одном `config.json`.
- **Мультиязычность** — легко добавлять новые языки интерфейса.
- **Логирование** — подробный лог всех шагов для отладки и аудита.
- **Отчеты** — экспорт в различные форматы для интеграции с другими системами.

## Где можно использовать

- **Агрегация и мониторинг крипто- и IT-проектов**
- **Автоматизация сбора контактных данных**
- **Обновление витрин и агрегаторов проектов**
- **Парсинг публичных профилей и документации**

## Технологический стек

- **Python** — основной язык разработки
- **Requests, BeautifulSoup** — парсинг и обработка сайтов
- **Playwright** — парсинг X-профилей (через fingerprint)

### Поддерживаемые источники

| Источник          | Описание                                  |
|-------------------|-------------------------------------------|
| `website`         | Главный сайт проекта                      |
| `docs`            | Документация, whitepaper                  |
| `X/Twitter`       | Bio и линки профиля, аватар               |
| `linktr.ee`/etc.  | Сбор всех связанных соцсетей              |
| `YouTube`         | Корректная агрегация только каналов       |
| `GitHub`          | Поддержка фильтрации только по org/user   |

## Архитектура

### Компоненты системы

1. **Парсеры (`core/*.py`)** — обертки над разными источниками (сайты, коллекционные сервисы, X/Twitter).
2. **Центральная точка входа (`config/start.py`)** — управляет пайплайном сбора, нормализации и сохранения данных.
3. **Шаблоны (`templates/`)** — структуруют результат под формат системы.
4. **Логирование (`logs/`)** — ведет полный журнал работы.
5. **Конфигурация (`config/config.json`)** — все цели, настройки и параметры.

### Структура проекта

strapiprojectsbot/
├── config/
│   ├── apps/
│   │   ├── {project}.json         # Пример конфига отдельного проекта/приложения
│   ├── config.json                # Центральная конфигурация (все проекты, глобальные параметры)
│   ├── start.py                   # Главный скрипт пайплайна (точка входа)
├── core/
│   ├── api_ai.py                  # Модуль интеграции с AI
│   ├── api_strapi.py              # Модуль интеграции с API Strapi
│   ├── browser_fetch.js           # Скрипт для парсинга сайтов через браузер
│   ├── install.py                 # Скрипт автоустановки зависимостей
│   ├── package.json               # Описание зависимостей для парсеров
│   ├── package-lock.json          # Лок-файл зависимостей node_modules
│   ├── twitter_parser.js          # Скрипт для парсинга X профилей и ссылок
│   ├── web_parser.py              # Модуль парсинга и нормализации ссылок
├── logs/
│   ├── ai.log                     # ИИ лог пайплайна
│   ├── host.log                   # Хостовой лог пайплайна
│   ├── setup.log                  # Лог установки зависимостей
│   ├── strapi.log                 # Лог отправки в Strapi
├── storage/
│   └── apps/
│       └── {project}/
│           └── main.json          # Результаты парсинга по каждому проекту
├── templates/
│   └── main_template.json         # Шаблон структуры проекта для заполнения
├── requirements.txt               # Python зависимости
├── README.md                      # Документация на английском
└── start.sh                       # Скрипт для быстрого запуска всего пайплайна

## Pipeline: Как это работает?

1. **Система стартует через** `start.sh`, который запускает `config/start.py`.
2. **Установка зависимостей**:
   * `config/start.py` автоматически запускает `core/install.py` — происходит установка Python-пакетов, Node.js-зависимостей и загрузка браузеров Playwright.
3. **Загрузка конфигурации**:
   * Загружается основной конфиг и шаблон структуры проекта.
4. **Парсинг каждой цели** (в конфиге):
   * **Сначала** бот пробует собрать данные через обычный парсинг (`requests` + `BeautifulSoup`).
   * **Если сайт защищён (антибот/Cloudflare/JS)** — автоматически используется браузерный парсер с Fingerprint Suite (`core/browser_fetch.js`, Playwright + fingerprint-injector).
   * **Twitter/X** всегда обрабатывается отдельно через `core/twitter_parser.js` (Playwright + Fingerprint Suite).
   * Внутренние страницы, docs, коллекционные сервисы (`linktr.ee` и др.) проходят тем же пайплайном: сначала requests, при необходимости через Fingerprint Suite.
   * Детектируются все соцсети, docs, коллекционные ссылки, логотипы.
   * Все данные проходят нормализацию (YouTube-канал, docs, GitHub, Medium).
5. **Сохранение результата**:
   * Данные сохраняются в `storage/total/{project}/main.json`.
6. **Формирование и публикация отчета**:
   * Собранные данные автоматически могут быть загружены в Strapi через API для дальнейшей публикации.

**Весь процесс полностью автоматизирован — запуск `start.sh` гарантирует установку зависимостей и корректный обход антибот-защиты (Cloudflare, JS) с помощью Fingerprint Suite, без ручного вмешательства.**

## Установка и запуск

```bash
git clone https://github.com/beesyst/strapiprojectsbot.git
cd strapiprojectsbot
bash start.sh
```

## Настройка конфигурации
Все параметры задаются в файле config/config.json:

| Параметр   | Значение по умолчанию | Описание                                                     |
|------------|-----------------------|--------------------------------------------------------------|
| `apps`     | `[ "babylon" ]`       | Список целей (объекты-проекты с настройками и enabled)       |
| `enabled`  | `true`                | Флаг: включен ли проект (false — будет полностью проигнорирован) |
| `link_collections` | `[ "linktr.ee" ]` | Массив сервисов для глубокого парсинга                 |

## Терминал и статусы

В процессе работы бот выводит для каждого проекта только итоговый статус:

- `[add]` — проект добавлен впервые (создан новый main.json, отправлен в Strapi)
- `[update]` — данные проекта обновлены (main.json перезаписан, отправлен в Strapi)
- `[skip]` — данные не изменились (ничего не отправлялось)
- `[error]` — возникла ошибка при сборе или отправке

