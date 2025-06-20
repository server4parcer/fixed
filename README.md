# YCLIENTS Parser

Система для автоматизированного сбора данных с платформы YCLIENTS. Позволяет парсить информацию о доступных датах бронирования, временных слотах, ценах, исполнителях и номерах мест. Включает инструменты для бизнес-аналитики рынка спортивных объектов.

## Содержание

- [Возможности](#возможности)
- [Архитектура](#архитектура)
- [Требования](#требования)
- [Установка](#установка)
- [Настройка](#настройка)
- [Использование](#использование)
- [API Документация](#api-документация)
- [Бизнес-аналитика](#бизнес-аналитика)
- [Разработка и тестирование](#разработка-и-тестирование)
- [Часто задаваемые вопросы](#часто-задаваемые-вопросы)

## Возможности

- **Парсинг данных с YCLIENTS**: Сбор информации о доступных датах, времени, ценах, исполнителях и местах.
- **Обход защиты от ботов**: Система использует Playwright с расширенными возможностями эмуляции поведения пользователя и ротацией прокси для избежания блокировок.
- **Автоматическое обновление**: Обновление данных каждые 10 минут (настраиваемый интервал).
- **Хранение данных**: Данные сохраняются в PostgreSQL и доступны через Supabase.
- **Экспорт данных**: Возможность экспорта данных в форматах CSV и JSON.
- **REST API**: API для доступа к данным и управления парсером.
- **Контейнеризация**: Полная поддержка Docker для простого развертывания.
- **Бизнес-аналитика**: Расширенные возможности для анализа рынка спортивных объектов.

## Архитектура

Система построена на модульной архитектуре, что обеспечивает гибкость и масштабируемость:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Playwright │────▶│    Parser   │────▶│  Database   │
│  Browser    │     │   Module    │     │   Module    │
└─────────────┘     └─────────────┘     └─────────────┘
                           │                   │
                           ▼                   ▼
                    ┌─────────────┐     ┌─────────────┐
                    │   Export    │     │     API     │
                    │   Module    │     │   Module    │
                    └─────────────┘     └─────────────┘
                                              │
                                              ▼
                                        ┌─────────────┐
                                        │  Business   │
                                        │  Analytics  │
                                        └─────────────┘
```

## Требования

- Python 3.10+
- PostgreSQL 14+
- Docker и Docker Compose (опционально, для контейнеризации)

## Установка

### Установка с использованием Docker (рекомендуется)

1. Клонируйте репозиторий:
   ```bash
   git clone https://github.com/your-username/yclients-parser.git
   cd yclients-parser
   ```

2. Создайте файл .env на основе .env.example:
   ```bash
   cp .env.example .env
   ```

3. Настройте переменные окружения в файле .env

4. Запустите контейнеры с помощью Docker Compose:
   ```bash
   docker-compose up -d
   ```

### Ручная установка

1. Клонируйте репозиторий:
   ```bash
   git clone https://github.com/your-username/yclients-parser.git
   cd yclients-parser
   ```

2. Создайте и активируйте виртуальное окружение:
   ```bash
   python -m venv venv
   source venv/bin/activate  # Linux/macOS
   venv\Scripts\activate     # Windows
   ```

3. Установите зависимости:
   ```bash
   pip install -r requirements.txt
   ```

4. Установите Playwright и необходимые браузеры:
   ```bash
   playwright install chromium
   playwright install-deps chromium
   ```

5. Создайте файл .env на основе .env.example:
   ```bash
   cp .env.example .env
   ```

6. Настройте PostgreSQL и обновите переменные окружения в .env

7. Обновите схему базы данных для поддержки бизнес-аналитики:
   ```bash
   python scripts/update_db_schema.py
   ```

## Настройка

### Основные настройки

Настройки можно задать через переменные окружения в файле .env:

- `API_KEY` - Ключ API для доступа к API-серверу
- `PARSE_URLS` - Список URL для парсинга, разделенных запятыми
- `PARSE_INTERVAL` - Интервал обновления данных в секундах (по умолчанию 600)

### Настройки базы данных

- `DB_HOST` - Хост базы данных
- `DB_PORT` - Порт базы данных
- `DB_NAME` - Имя базы данных
- `DB_USER` - Пользователь базы данных
- `DB_PASSWORD` - Пароль базы данных

### Настройки Supabase (опционально)

- `SUPABASE_URL` - URL Supabase
- `SUPABASE_KEY` - Ключ доступа Supabase

### Настройки прокси (опционально)

- `PROXY_SERVERS` - Список серверов прокси, разделенных запятыми
- `PROXY_USERNAMES` - Список пользователей прокси, разделенных запятыми
- `PROXY_PASSWORDS` - Список паролей прокси, разделенных запятыми
- `PROXY_PORTS` - Список портов прокси, разделенных запятыми

Альтернативно, прокси можно настроить через файл `data/proxies.json`:

```json
[
    {
        "server": "proxy1.example.com",
        "username": "user1",
        "password": "pass1",
        "port": 8080
    },
    {
        "server": "proxy2.example.com",
        "username": "user2",
        "password": "pass2",
        "port": 8080
    }
]
```

## Использование

### Запуск парсера

Для запуска всех компонентов приложения:

```bash
python src/main.py --mode all
```

Для запуска только парсера:

```bash
python src/main.py --mode parser --urls https://yclients.com/company/111111/booking https://yclients.com/company/222222/booking
```

Для запуска только API-сервера:

```bash
python src/main.py --mode api
```

### Параметры командной строки

- `--mode`: Режим работы (`parser`, `api`, `all`)
- `--urls`: Список URL для парсинга
- `--once`: Запустить парсер только один раз (без непрерывного режима)
- `--interval`: Интервал обновления данных в секундах

### Через Docker Compose

```bash
docker-compose up -d
```

## API Документация

API-сервер доступен по адресу `http://localhost:8000`.

### Аутентификация

Все запросы к API должны содержать заголовок `X-API-Key` с API-ключом.

### Основные эндпоинты

- `GET /status` - Получение статуса парсера
- `GET /urls` - Получение списка URL для парсинга
- `POST /urls` - Создание нового URL для парсинга
- `GET /urls/{url_id}` - Получение информации о URL
- `PUT /urls/{url_id}` - Обновление информации о URL
- `DELETE /urls/{url_id}` - Удаление URL
- `GET /data` - Получение данных бронирования
- `GET /export` - Экспорт данных бронирования в файл
- `GET /download/{filename}` - Скачивание файла
- `POST /parse` - Запуск парсера для указанного URL
- `POST /parse/all` - Запуск парсера для всех URL
- `GET /analytics/pricing` - Анализ цен по различным критериям
- `GET /analytics/availability` - Анализ доступности по различным критериям
- `GET /analytics/price_history` - История изменения цен

### Swagger UI

API-документация доступна через Swagger UI по адресу `http://localhost:8000/docs`.

## Бизнес-аналитика

Модуль бизнес-аналитики предоставляет расширенные возможности для анализа рынка спортивных объектов.

### Обновление до версии с бизнес-аналитикой

Если вы обновляетесь с предыдущей версии, выполните следующие шаги:

1. Обновите схему базы данных:
   ```bash
   python scripts/update_db_schema.py
   ```

2. Перезапустите приложение.

### Расширенный сбор данных

Расширенный экстрактор данных собирает дополнительную информацию:

- **Определение типа корта**: Автоматически определяет типы кортов/площадок (теннис, баскетбол, сквош и т.д.)
- **Информация о местоположении**: Извлекает и нормализует данные о местоположении объекта
- **Категоризация времени**: Категоризирует бронирования как ДНЕВНОЕ, ВЕЧЕРНЕЕ или ВЫХОДНЫЕ
- **Требования предоплаты**: Определяет, требуется ли предоплата для бронирования
- **Популярность объекта**: Отслеживает количество отзывов и рейтинги

### Фильтрация и анализ

Используйте следующие фильтры с API и функциями экспорта:

#### Фильтрация через API

```
GET /data?court_type=tennis&time_category=EVENING&location=central
```

#### Аналитические отчеты

```
GET /analytics/pricing?court_type=tennis&time_frame=last_30_days
GET /analytics/availability?location=north&court_type=basketball
GET /analytics/price_history?venue_id=123&start_date=2023-01-01
```

### Понимание бизнес-отчетов

#### Анализ цен

Показывает средние, минимальные и максимальные цены для:
- Типов кортов/площадок
- Категорий времени (ДНЕВНОЕ, ВЕЧЕРНЕЕ, ВЫХОДНЫЕ)
- Местоположений
- Диапазонов дат

#### Тенденции доступности

Отслеживает закономерности доступности площадок:
- Временные слоты с высоким спросом
- Сезонные закономерности
- Спрос в будние и выходные дни
- Доступность по местоположению

#### Отслеживание цен конкурентов

- История цен по объектам
- Обнаружение изменений цен
- Идентификация специальных предложений
- Ценообразование премиум и стандартных площадок

### Экспорт данных бизнес-аналитики

Расширенные возможности экспорта:

```
GET /export/json?include_analytics=true&court_type=tennis
GET /export/csv?time_category=WEEKEND&location=west
```

## Разработка и тестирование

### Структура проекта

```
yclients-parser/
├── .env               # Переменные окружения
├── .gitignore        # Игнорируемые Git файлы
├── README.md         # Документация
├── requirements.txt  # Зависимости Python
├── docker-compose.yml # Конфигурация Docker Compose
├── Dockerfile        # Dockerfile для сборки образа
├── config/           # Конфигурация проекта
├── src/              # Исходный код
│   ├── api/          # API-сервер
│   ├── browser/      # Управление браузером
│   ├── database/     # Работа с базой данных
│   ├── export/       # Экспорт данных
│   ├── parser/       # Парсер YCLIENTS
│   └── scheduler/    # Планировщик задач
├── tests/            # Тесты
├── data/             # Данные проекта
└── logs/             # Логи проекта
```

### Запуск тестов

```bash
python tests/run_tests.py
```

### Примеры использования API для бизнес-аналитики

#### Получение аналитики цен

```python
import requests

api_url = "http://localhost:8000"
headers = {"X-API-Key": "your-api-key"}

# Получение аналитики цен для теннисных кортов в вечернее время
response = requests.get(
    f"{api_url}/analytics/pricing",
    params={"court_type": "tennis", "time_category": "EVENING"},
    headers=headers
)
pricing_data = response.json()
print(pricing_data)
```

#### Экспорт данных с фильтрацией

```python
import requests

api_url = "http://localhost:8000"
headers = {"X-API-Key": "your-api-key"}

# Экспорт данных в формате JSON с фильтрацией по типу корта и категории времени
response = requests.get(
    f"{api_url}/export/json",
    params={"court_type": "tennis", "time_category": "WEEKEND", "include_analytics": "true"},
    headers=headers
)

with open("tennis_weekend_data.json", "wb") as f:
    f.write(response.content)
```

## Часто задаваемые вопросы

### Как добавить новые URL для парсинга?

Вы можете добавить новые URL через API или напрямую в файл `data/urls.txt` (по одному URL на строку).

### Как настроить прокси?

Прокси можно настроить через переменные окружения или файл `data/proxies.json`. См. раздел [Настройки прокси](#настройки-прокси-опционально).

### Как изменить интервал обновления данных?

Интервал обновления данных можно изменить через переменную окружения `PARSE_INTERVAL` или параметр командной строки `--interval`.

### Что делать, если парсер блокируется?

Если парсер блокируется, попробуйте:
1. Использовать прокси-серверы
2. Увеличить интервал обновления данных
3. Уменьшить количество одновременных запросов

### Как экспортировать данные?

Данные можно экспортировать через API или с помощью функций `export_to_csv()` и `export_to_json()` из модуля `DatabaseManager`.

### Как использовать аналитику для принятия бизнес-решений?

- **Мониторинг цен конкурентов**: Используйте эндпоинт `/analytics/pricing` для мониторинга цен конкурентов в разное время суток и по разным типам площадок.
- **Оптимизация расписания**: Анализируйте данные доступности через `/analytics/availability` для определения наиболее востребованных временных слотов.
- **Динамическое ценообразование**: Используйте данные о сезонных трендах и пиковых часах для настройки цен.
- **Конкурентный анализ**: Сравнивайте цены и доступность ваших объектов с конкурентами по местоположению и типу корта.

## Лицензия

Этот проект распространяется под лицензией MIT. См. файл LICENSE для получения подробной информации.