# <Назва сервісу>

Одне речення: яку проблему вирішує сервіс і для кого.

## Можливості

- <Ключова можливість 1>
- <Ключова можливість 2>

## Архітектура

Коротко опишіть межі сервісу, основні процеси та зовнішні залежності.

```text
Client / Backoffice → API → PostgreSQL
                         → Redis / BullMQ → Worker
                         → S3-compatible storage
```

Не дублюйте детальну документацію модулів. Додайте на неї посилання.

## Вимоги

- Docker і Docker Compose v2
- Node.js <версія>
- pnpm <версія>

## Швидкий старт

```bash
cp .env.example .env
docker compose up --build
```

Після запуску:

- API: `http://localhost:<port>/api/v1`
- Swagger: `http://localhost:<port>/docs`
- Health: `http://localhost:<port>/api/v1/health/ready`

## Конфігурація

| Змінна       | Обов'язкова | Опис       | Приклад без секретів  |
| ------------ | ----------: | ---------- | --------------------- |
| `NODE_ENV`   |         Так | Середовище | `development`         |
| `<VARIABLE>` |      Так/Ні | <Опис>     | `<безпечний приклад>` |

Секрети не зберігаються в git. Вкажіть, де їх налаштовувати для production.

## Команди

```bash
pnpm dev
pnpm lint
pnpm typecheck
pnpm test
pnpm test:e2e
```

## База даних і міграції

```bash
pnpm prisma migrate dev
pnpm prisma migrate deploy
```

Поясніть різницю між локальною та production-командою.

## API й контракти

- OpenAPI: `<URL або шлях>`
- API prefix: `/api/v1`
- Auth: `<короткий опис>`
- Стабільні коди помилок: `<посилання>`

## Черги та scheduler

Опишіть назви черг, процеси worker/scheduler, retry/backoff та спосіб локальної перевірки.

## Тестування

Опишіть рівні тестів, необхідні залежності та точні команди.

## Спостережуваність

- Логи: <формат і correlation ID>
- Метрики: <endpoint або система>
- Error tracking: <система>
- Health endpoints: <перелік>

## Deploy і rollback

Опишіть порядок migrations, запуск процесів, readiness-перевірку та rollback.

## Типові проблеми

### <Симптом>

Причина: <коротко>.

Рішення:

```bash
<безпечна команда>
```

## Документація

- SDD workflow: `.docs/SDD_WORKFLOW.md`
- Правила агентів: `.docs/AGENT_RULES.md`
- <Посилання на архітектуру, runbooks і API>

## Внесок у проєкт

Перед зміною поведінки створіть спеку й план за правилами в `.docs`.
