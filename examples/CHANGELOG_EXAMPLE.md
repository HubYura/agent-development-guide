# Changelog: readiness-перевірка залежностей

- **Дата:** 2026-08-05
- **Версія:** `Unreleased`
- **Спека:** [SPEC-2026-001](./SPEC_EXAMPLE.md)
- **План:** [PLAN-2026-001](./PLAN_EXAMPLE.md)

## Added

- Додано `GET /api/v1/health/ready`, який повідомляє готовність PostgreSQL і Redis.
- Readiness response містить окремі статуси `postgres` і `redis` без технічних деталей підключення.
- OpenAPI описує відповіді `200` і `503`.

## Changed

- Docker Compose перевіряє готовність API через `/api/v1/health/ready`, а не лише роботу процесу.
- `/api/v1/health/live` залишається liveness-перевіркою й не залежить від PostgreSQL або Redis.

## Migration

Додаткові міграції даних або env-змінні не потрібні.

Після оновлення перевірити:

```bash
docker compose up -d --build
docker compose ps
curl --fail http://localhost:3000/api/v1/health/ready
```

Очікування: контейнер API має статус `healthy`, endpoint повертає `200` і обидва статуси `up`.

## Rollback

1. Повернути healthcheck на `/api/v1/health/live`.
2. Застосувати попередній Docker image.
3. Перевірити `docker compose ps` і liveness endpoint.

## Verification

```bash
pnpm lint
pnpm typecheck
pnpm test -- health
pnpm test:e2e -- health.e2e-spec.ts
docker compose config
```

Очікуваний результат: усі команди завершуються з кодом `0`.
