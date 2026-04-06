# ACID-DATING

Телеграм-бот знакомств для нишевого сообщества. Анкеты, лайки, геолокация, аудиотреки, админ-панель с рассылкой.

**Стек:** Python 3.12 · aiogram 3.x · PostgreSQL · Redis · Docker Compose

---

## Быстрый старт (Docker, polling)

### 1. Клонировать репозиторий

```bash
git clone https://gitlab.hoebbels.cfd/root/shizobot.git
cd shizobot
```

### 2. Создать `.env`

```bash
cp .env.example .env
```

Заполните `.env` (минимум для polling — только `BOT_TOKEN`):

```env
BOT_TOKEN=your-bot-token
BOT_MODE=polling

REDIS_HOST=redis
REDIS_PORT=6379

REQUIRED_CHATS=          # оставьте пустым, чтобы отключить проверку подписки

GEOCODER_APIKEY=         # ключ Яндекс.Карт (опционально)
```

> Полный список переменных — в `.env.example`.

### 3. Запустить

```bash
docker compose up -d
```

Миграции применяются автоматически при старте контейнера `bot`.

### 4. Добавить администратора

```bash
docker compose exec db psql -U admin -d shizobot -c "INSERT INTO admins VALUES (<ваш_telegram_id>);"
```

---

## Webhook (продакшн)

### 1. Настроить nginx и SSL

Получите SSL-сертификат (например, через Certbot), настройте nginx для проксирования вебхука:

```nginx
location /webhook {
    proxy_pass http://bot:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

Готовый `nginx.conf` есть в корне репозитория.

### 2. Заполнить webhook-переменные в `.env`

```env
BOT_MODE=webhook
BASE_WEBHOOK_URL=https://yourdomain.ru
WEBHOOK_PATH=/webhook
WEBHOOK_SECRET=your-secret
WEB_SERVER_HOST=0.0.0.0
WEB_SERVER_PORT=8080
```

### 3. Запустить

```bash
docker compose up -d
```

---

## Переменные окружения

| Переменная | Обязательна | Описание |
|---|---|---|
| `BOT_TOKEN` | да | Токен бота от @BotFather |
| `BOT_MODE` | да | `polling` (дев) или `webhook` (прод) |
| `REDIS_HOST` | да | Хост Redis (в Docker — `redis`) |
| `REDIS_PORT` | да | Порт Redis (по умолчанию `6379`) |
| `REQUIRED_CHATS` | нет | Каналы/группы через запятую (`-100XXX` или `@username`). Пусто — проверка отключена |
| `GEOCODER_APIKEY` | нет | API-ключ [Яндекс.Карт](https://developer.tech.yandex.ru) |
| `BASE_WEBHOOK_URL` | webhook | URL сервера (`https://yourdomain.ru`) |
| `WEBHOOK_PATH` | webhook | Путь вебхука (например `/webhook`) |
| `WEBHOOK_SECRET` | webhook | Секретный токен вебхука |
| `WEB_SERVER_HOST` | webhook | Хост веб-сервера (`0.0.0.0`) |
| `WEB_SERVER_PORT` | webhook | Порт веб-сервера (`8080`) |

---

## Полезные команды

```bash
# Логи бота
docker compose logs -f bot

# Остановить
docker compose down

# Остановить и удалить данные БД и Redis
docker compose down -v

# Применить миграции вручную
docker compose exec bot alembic upgrade head
```

---

## Команды бота

| Команда | Описание |
|---|---|
| `/start` | Регистрация или главное меню |
| `/menu` | Главное меню (только для зарегистрированных) |
| `/admin` | Панель администратора |
| `/ban <id\|@username>` | Бан пользователя (только для админов) |
| `/unban <id\|@username>` | Разбан пользователя (только для админов) |
