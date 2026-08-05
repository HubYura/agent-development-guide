---
id: PLAN-2026-001
spec: ./SPEC_EXAMPLE.md
spec_revision: 1
status: Done
owner: Backend agent
created: 2026-08-05
updated: 2026-08-05
---

# План: readiness-перевірка PostgreSQL і Redis

## Мета

Реалізувати й перевірити `/api/v1/health/live` та `/api/v1/health/ready` відповідно до `SPEC-2026-001`.

## Межі

**Входить:** HTTP endpoints, PostgreSQL/Redis indicators, timeout, OpenAPI, Compose healthcheck, тести й документація.

**Не входить:** MinIO, зовнішні providers, dashboard та alerts.

## Залежності й рішення

- Спека: [SPEC-2026-001](./SPEC_EXAMPLE.md), revision 1.
- Використовуються наявні Prisma та Redis clients; нові connection pools не створюються.
- HTTP-контракт не повертає текст внутрішніх помилок.

## Карта файлів

| Дія    | Файл                                                   | Відповідальність                                     |
| ------ | ------------------------------------------------------ | ---------------------------------------------------- |
| Create | `apps/api/src/health/health.controller.ts`             | HTTP endpoints і status codes                        |
| Create | `apps/api/src/health/health.service.ts`                | Паралельний запуск indicators і агрегація результату |
| Create | `apps/api/src/health/indicators/postgres.indicator.ts` | `SELECT 1` через Prisma                              |
| Create | `apps/api/src/health/indicators/redis.indicator.ts`    | Redis `PING`                                         |
| Create | `apps/api/src/health/health.module.ts`                 | Wiring залежностей                                   |
| Modify | `apps/api/src/app.module.ts`                           | Підключення `HealthModule`                           |
| Modify | `compose.yaml`                                         | API healthcheck                                      |
| Modify | `README.md`                                            | Документація endpoints                               |
| Test   | `apps/api/src/health/health.service.spec.ts`           | Unit-сценарії status aggregation                     |
| Test   | `test/health.e2e-spec.ts`                              | HTTP-контракт                                        |

## Кроки

### 1. Зафіксувати контракти й unit-поведінку

- [x] Створити типи `DependencyStatus = 'up' | 'down'` і `ReadinessResult` у `health.service.ts`.
- [x] Додати unit-тести:
  - обидві залежності `up` → загальний статус `ok`;
  - одна залежність `down` → статус `error`;
  - indicator кидає помилку → відповідна залежність `down`;
  - indicator не відповідає за 1 секунду → `down`.
- [x] Запустити:

```bash
pnpm test -- health.service.spec.ts
```

Очікування: до реалізації тести падають; після реалізації — проходять.

### 2. Реалізувати indicators

- [x] У `postgres.indicator.ts` виконати `prisma.$queryRaw\`SELECT 1\``через наявний`PrismaService`.
- [x] У `redis.indicator.ts` виконати `redis.ping()` через наявний `RedisService`.
- [x] Не створювати клієнти або підключення всередині indicators.
- [x] Не повертати сирі помилки з indicators.
- [x] Запустити unit та integration тести health-модуля.

### 3. Реалізувати HTTP endpoints

- [x] `GET /api/v1/health/live` завжди повертає `{ "status": "ok" }`, якщо HTTP-процес працює.
- [x] `GET /api/v1/health/ready` запускає indicators паралельно.
- [x] Для `error` установити HTTP status `503`.
- [x] Додати Swagger response schemas для `200` і `503`.
- [x] Перевірити, що response не містить exception message або stack trace.

### 4. Додати e2e-сценарії

- [x] Перевірити контракт `live`.
- [x] Перевірити readiness за справних залежностей.
- [x] Замінити Redis indicator тестовим adapter і перевірити `503`.
- [x] Замінити PostgreSQL indicator тестовим adapter і перевірити `503`.
- [x] Запустити:

```bash
pnpm test:e2e -- health.e2e-spec.ts
```

### 5. Оновити Compose й документацію

- [x] У `compose.yaml` змінити API healthcheck на:

```yaml
healthcheck:
  test: ['CMD', 'wget', '--spider', '-q', 'http://localhost:3000/api/v1/health/ready']
  interval: 10s
  timeout: 2s
  retries: 5
  start_period: 20s
```

- [x] Оновити README та OpenAPI.
- [x] Додати changelog.

## Міграція, rollout і rollback

Схема даних не змінюється.

1. Deploy API з новими endpoints.
2. Перевірити `live` і `ready` вручну.
3. Deploy Compose healthcheck.
4. Перевірити, що контейнер `healthy`, а restart count не збільшується.

Rollback: повернути healthcheck на `/api/v1/health/live` і відкотити image. Дані не потребують відновлення.

## Команди фінальної перевірки

```bash
pnpm format:check
pnpm lint
pnpm typecheck
pnpm test -- health
pnpm test:e2e -- health.e2e-spec.ts
docker compose config
docker compose up -d --build
curl --fail http://localhost:3000/api/v1/health/live
curl --fail http://localhost:3000/api/v1/health/ready
```

## Відповідність критеріям

| Критерій спеки | Крок | Доказ                                          |
| -------------- | ---- | ---------------------------------------------- |
| AC-1           | 2–4  | e2e: `returns 200 when dependencies are up`    |
| AC-2           | 2–4  | e2e: `returns 503 when redis is down`          |
| AC-3           | 2–4  | e2e: `returns 503 when postgres is down`       |
| AC-4           | 3–4  | snapshot error response без внутрішніх деталей |
| AC-5           | 5    | `docker compose config` і healthy status       |
| AC-6           | 3, 5 | OpenAPI snapshot і README diff                 |

## Відхилення від плану

| Дата       | Відхилення                                                    | Причина                          | Вплив                                                                        |
| ---------- | ------------------------------------------------------------- | -------------------------------- | ---------------------------------------------------------------------------- |
| 2026-08-05 | Для e2e використано test adapters замість зупинки контейнерів | Тести мають бути детермінованими | Контракт і деградація перевірені; ручна Compose-перевірка залишена у rollout |

## Завершення

- [x] Усі критерії приймання мають докази.
- [x] Definition of Done виконано.
- [x] Спека, план, changelog і README узгоджені.
