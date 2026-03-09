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
   URL вебхуков в воркфлоу должны быть вида `https://ВАШ_ДОМЕН/webhook/...` (то же значение, что в `WEBHOOK_URL` в `.env`).

**Важно в `.env`:** при доступе через Caddy указывайте домен без порта 5678: `N8N_HOST=домен`, `N8N_EDITOR_BASE_URL=https://домен`, `WEBHOOK_URL=https://домен`. Переменная `N8N_DOMAIN` — hostname для Caddy (без порта); если не задана, подставляется `N8N_HOST`. При использовании нестандартного HTTPS-порта (например 8443) задайте `N8N_DOMAIN=домен` без порта, иначе Caddy может не запуститься.