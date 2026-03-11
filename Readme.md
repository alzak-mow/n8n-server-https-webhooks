Структура проекта

n8n-docker/
├── .env
├── docker-compose.yml
├── caddy/
│   └── Caddyfile.server
├── data/
│   └── postgres/
├── n8n/
│   └── demo-data/
│       ├── credentials/
│       └── workflows/
└── shared/


1. Клонируем структуру проекта

mkdir -p n8n-docker/{caddy,data/postgres,n8n/demo-data/{credentials,workflows},shared}
cd n8n-docker


# Генерация случайных ключей (опционально)
openssl rand -hex 16  # Для N8N_ENCRYPTION_KEY
openssl rand -hex 32  # Для N8N_USER_MANAGEMENT_JWT_SECRET

---

## Проверка конфигурации после деплоя

1. **Контейнеры и здоровье**
   ```bash
   docker compose ps
   ```
   Все сервисы (postgres, n8n, caddy) должны быть `Up` и (где есть) `healthy`.

2. **Логи Caddy**
   ```bash
   docker compose logs caddy --tail 50
   ```
   Не должно быть ошибок TLS/ACME; при первом запросе к домену появится получение сертификата Let's Encrypt.

3. **Доступ к n8n**
   - В браузере: `https://ВАШ_ДОМЕН` (или `https://ВАШ_ДОМЕН:8443`, если задан `CADDY_HTTPS_PORT=8443`).
   - Health n8n через Caddy: `curl -k https://ВАШ_ДОМЕН/healthz` — должен вернуть 200.

4. **Webhooks**
   URL вебхуков в воркфлоу должны быть вида `https://ВАШ_ДОМЕН/webhook/...` (то же значение, что в `WEBHOOK_URL` в `.env`). Чтобы **Test URL** и **Production URL** в ноде Webhook совпадали, задайте в `.env` одинаковые `N8N_EDITOR_BASE_URL` и `WEBHOOK_URL` (или только `N8N_EDITOR_BASE_URL` — тогда `WEBHOOK_URL` подставится таким же).

**Важно в `.env`:** при доступе через Caddy указывайте домен без порта 5678: `N8N_HOST=домен`, `N8N_EDITOR_BASE_URL=https://домен`, `WEBHOOK_URL=https://домен`. Переменная `N8N_DOMAIN` — hostname для Caddy (без порта); если не задана, подставляется `N8N_HOST`. При использовании нестандартного HTTPS-порта (например 8443) задайте `N8N_DOMAIN=домен` без порта, иначе Caddy может не запуститься.

---

## Ошибка «This site cannot provide a secure connection» / «sent an invalid response» (порт 8443)

**Причина:** Let's Encrypt выдаёт сертификаты только после проверки по **порту 80** (HTTP-01). Если Caddy слушает только 8080 и 8443, проверка не проходит, сертификат не выдаётся (или используется самоподписанный) — браузер и клиенты не доверяют соединению.

**Что сделать:**

1. **Освободить порт 80** на сервере и дать Caddy слушать 80 и 443:
   ```bash
   sudo ss -tlnp | grep :80   # что заняло 80
   sudo systemctl stop nginx  # или apache2, или другой сервис
   ```
   В `.env` закомментируйте или удалите строки:
   ```bash
   # CADDY_HTTP_PORT=8080
   # CADDY_HTTPS_PORT=8443
   ```
   Перезапустите стек:
   ```bash
   docker compose down && docker compose up -d
   ```
   В `.env` переведите URL на стандартный HTTPS без порта:
   ```bash
   N8N_HOST=xqyx.ru
   N8N_EDITOR_BASE_URL=https://xqyx.ru
   WEBHOOK_URL=https://xqyx.ru
   ```
   После этого доступ и вебхуки: **https://xqyx.ru** (без :8443). Caddy получит сертификат Let's Encrypt, ошибка «secure connection» исчезнет.

2. **Если порт 80 освободить нельзя** — нужна проверка домена через DNS (DNS-01). Это делается отдельной сборкой Caddy с DNS-провайдером и настройкой в Caddyfile (например, Cloudflare API). Для типичного деплоя проще освободить порт 80.