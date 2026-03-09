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