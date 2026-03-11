# n8n-server

Готовый стек для развёртывания [n8n](https://n8n.io) с PostgreSQL и Caddy: автоматический HTTPS (Let's Encrypt), хранение данных в БД и на диске.

## Содержимое

- **n8n** — оркестрация воркфлоу (последний образ)
- **PostgreSQL 16** — база данных
- **Caddy 2** — reverse proxy, TLS из коробки, HTTP/3

## Требования

- Docker и Docker Compose (v2+)
- Домен с DNS, указывающим на ваш сервер (для Let's Encrypt)
- Порты 80 и 443 свободны (или настраиваемые через переменные)

## Структура проекта

```
n8n-server/
├── .env                 # Ваши настройки (не коммитить)
├── .env-example         # Шаблон переменных окружения
├── docker-compose.yml   # Сервисы: postgres, n8n, n8n-import, caddy
├── caddy/
│   └── Caddyfile.server # Конфиг Caddy (домен из .env)
├── data/
│   └── postgres/        # Данные PostgreSQL (создаётся при первом запуске)
├── n8n/
│   └── demo-data/       # Опционально: credentials/, workflows/ для импорта
│       ├── credentials/
│       └── workflows/
└── shared/              # Общая папка для файлов воркфлоу
```

## Быстрый старт

### 1. Клонирование и настройка окружения

```bash
git clone https://github.com/alzak-mow/n8n-server-https-webhooks
cd n8n-server
cp .env-example .env
```

### 2. Редактирование `.env`

Обязательно задайте:

- `POSTGRES_PASSWORD` — пароль БД
- `N8N_ENCRYPTION_KEY` и `N8N_USER_MANAGEMENT_JWT_SECRET` — секреты n8n
- `N8N_DOMAIN`, `N8N_HOST`, `N8N_EDITOR_BASE_URL`, `WEBHOOK_URL` — ваш домен (см. раздел «Переменные окружения»)

Генерация случайных ключей:

```bash
openssl rand -hex 16   # для N8N_ENCRYPTION_KEY
openssl rand -hex 32   # для N8N_USER_MANAGEMENT_JWT_SECRET
```

### 3. Запуск

```bash
docker compose up -d
```

Проверка статуса:

```bash
docker compose ps
```

Все сервисы должны быть в состоянии `Up`, postgres и n8n — `healthy`.

## Переменные окружения

Файл `.env` задаёт настройки стека. Ниже список переменных (по образцу `.env-example`).

| Переменная | Описание | Пример |
|------------|----------|--------|
| **PostgreSQL** | | |
| `POSTGRES_DATA_PATH` | Путь к данным БД на хосте | `./data/postgres` |
| `POSTGRES_USER` | Пользователь PostgreSQL | `n8n_user` |
| `POSTGRES_PASSWORD` | Пароль PostgreSQL | задать свой |
| `POSTGRES_DB` | Имя базы | `n8n_db` |
| **n8n** | | |
| `N8N_ENCRYPTION_KEY` | Ключ шифрования (32 hex-символа) | сгенерировать |
| `N8N_USER_MANAGEMENT_JWT_SECRET` | Секрет JWT (64 hex-символа) | сгенерировать |
| `N8N_DOMAIN` | Домен для Caddy (только hostname) | `example.com` |
| `N8N_HOST` | Домен (с портом, если не 443) | `example.com` или `example.com:8443` |
| `N8N_EDITOR_BASE_URL` | URL редактора n8n | `https://example.com` |
| `WEBHOOK_URL` | Базовый URL для webhook | `https://example.com` |
| `N8N_SECURE_COOKIE` | Безопасные cookie (для HTTPS — `true`) | `true` или `false` |
| **Caddy** | | |
| `CADDY_HTTP_PORT` | Порт HTTP на хосте (опционально) | `80` (по умолчанию) |
| `CADDY_HTTPS_PORT` | Порт HTTPS на хосте (опционально) | `443` (по умолчанию) |
| **Прочее** | | |
| `TZ` | Часовой пояс | `Europe/Moscow` |

Если порты 80/443 заняты, задайте, например: `CADDY_HTTP_PORT=8080`, `CADDY_HTTPS_PORT=8443` и в n8n-переменных укажите домен с портом: `N8N_HOST=example.com:8443`, `N8N_EDITOR_BASE_URL=https://example.com:8443`, `WEBHOOK_URL=https://example.com:8443`.

## Проверка после деплоя

1. **Контейнеры и здоровье**

   ```bash
   docker compose ps
   ```
   Все сервисы (postgres, n8n, caddy) — `Up`, где есть healthcheck — `healthy`.

2. **Логи Caddy**

   ```bash
   docker compose logs caddy --tail 50
   ```
   Не должно быть ошибок TLS/ACME; при первом заходе на домен Caddy получит сертификат Let's Encrypt.

3. **Доступ к n8n**

   В браузере: `https://ВАШ_ДОМЕН` (или `https://ВАШ_ДОМЕН:8443`, если используете нестандартный порт).

   Проверка health через Caddy:

   ```bash
   curl -k https://ВАШ_ДОМЕН/healthz
   ```
   Ожидается ответ с кодом 200.

4. **Webhooks**

   URL вебхуков в воркфлоу должны быть вида `https://ВАШ_ДОМЕН/webhook/...` (то же значение, что в `WEBHOOK_URL`). Чтобы Test URL и Production URL в ноде Webhook совпадали, задайте в `.env` одинаковые `N8N_EDITOR_BASE_URL` и `WEBHOOK_URL`.

## Лицензия

Используйте по своему усмотрению. n8n, PostgreSQL и Caddy распространяются под своими лицензиями.
