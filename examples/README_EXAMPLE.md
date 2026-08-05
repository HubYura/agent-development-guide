# lunch-core

Backend сервіс платформи «Обід»: API, фонові задачі та інтеграції для організації шкільного харчування.

> Це приклад README. Команди й файли потрібно синхронізувати з реальною реалізацією репозиторію.

## Можливості

- REST API для клієнтського Next.js-застосунку й окремого Next.js-бекофісу.
- PostgreSQL для транзакційних даних.
- Redis і BullMQ для фонових задач та scheduler.
- OpenAPI як публічний контракт API.

## Архітектура

`lunch-core` — модульний моноліт із трьома runtime-процесами з одного image:

```text
Client Next.js ─────┐
                    ├→ NestJS API → PostgreSQL
Backoffice Next.js ─┘            → Redis / BullMQ → Worker
                                              └───→ Scheduler
```

API, worker і scheduler мають спільні доменні модулі, але окремі entrypoints. Зовнішні providers підключаються через adapters.

## Вимоги

- Docker 27+
- Docker Compose v2
- Node.js 24 LTS
- pnpm через Corepack

Для звичайного локального запуску достатньо Docker і Compose.

## Швидкий старт

```bash
cp .env.example .env
docker compose up --build
```

Після запуску:

- API: `http://localhost:3000/api/v1`
- Swagger: `http://localhost:3000/docs`
- Liveness: `http://localhost:3000/api/v1/health/live`
- Readiness: `http://localhost:3000/api/v1/health/ready`

Перевірка:

```bash
curl --fail http://localhost:3000/api/v1/health/ready
```

Очікувана відповідь:

```json
{
  "status": "ok",
  "checks": {
    "postgres": "up",
    "redis": "up"
  }
}
```

## Конфігурація

| Змінна                  | Обов'язкова | Опис                      | Локальний приклад         |
| ----------------------- | ----------: | ------------------------- | ------------------------- |
| `NODE_ENV`              |         Так | Середовище                | `development`             |
| `PORT`                  |         Так | HTTP-порт API             | `3000`                    |
| `DATABASE_URL`          |         Так | PostgreSQL connection URL | Значення з `.env.example` |
| `REDIS_HOST`            |         Так | Host Redis                | `redis`                   |
| `REDIS_PORT`            |         Так | Port Redis                | `6379`                    |
| `CLIENT_APP_ORIGIN`     |         Так | CORS origin клієнта       | `http://localhost:3001`   |
| `BACKOFFICE_APP_ORIGIN` |         Так | CORS origin бекофісу      | `http://localhost:3002`   |

Секрети зберігаються поза git. Production-значення налаштовуються в secret manager платформи розгортання.

## Основні команди

```bash
pnpm dev
pnpm format:check
pnpm lint
pnpm typecheck
pnpm test
pnpm test:e2e
```

Docker Compose:

```bash
docker compose up -d --build
docker compose logs -f api worker scheduler
docker compose ps
docker compose down
```

## База даних і міграції

Локальна міграція під час розробки:

```bash
pnpm prisma migrate dev --name add-order-status
```

Застосування вже створених міграцій у deploy-процесі:

```bash
pnpm prisma migrate deploy
```

Не запускайте `migrate dev` у production.

## API й помилки

- Prefix: `/api/v1`
- OpenAPI UI: `/docs`
- JSON schema: `/docs-json`
- Кожна error response має стабільний `code` і `requestId`.
- Внутрішні exception messages і stack traces не повертаються клієнту.

## Health endpoints

| Endpoint               | Призначення                        | Успіх               |
| ---------------------- | ---------------------------------- | ------------------- |
| `/api/v1/health/live`  | Node.js-процес приймає HTTP-запити | `200`               |
| `/api/v1/health/ready` | PostgreSQL і Redis готові          | `200`; інакше `503` |

Readiness використовується балансувальником. Liveness не слід прив'язувати до коротких збоїв зовнішніх залежностей.

## Тестування

```bash
pnpm test          # unit та integration
pnpm test:e2e      # HTTP contracts
```

Тести не звертаються до production services. Зовнішні SMS і payment providers замінюються test adapters.

## Спостережуваність

- Структуровані JSON-логи через Pino.
- Кожен HTTP-запит має `requestId`.
- JWT, OTP, cookies і credentials маскуються.
- Помилки зовнішніх залежностей містять назву adapter, duration і безпечний error code.

## Типові проблеми

### API має статус `unhealthy`

Перевірте залежності:

```bash
docker compose ps
docker compose logs postgres redis api
curl -i http://localhost:3000/api/v1/health/ready
```

Не видаляйте volumes, доки не переконалися, що локальні дані не потрібні.

## Документація

- [SDD workflow](../SDD_WORKFLOW.md)
- [Правила агентної розробки](../AGENT_RULES.md)
- [Контрольний список якості](../QUALITY_CHECKLIST.md)

## Внесок у проєкт

Для зміни поведінки:

1. створіть спеку з `.docs/templates/SPEC_TEMPLATE.md`;
2. погодьте її;
3. створіть план з `.docs/templates/PLAN_TEMPLATE.md`;
4. реалізуйте й надайте докази критеріїв приймання;
5. оновіть changelog і цей README за потреби.
