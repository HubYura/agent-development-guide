---
task: LUNCH-1
spec: ./SPEC_EXAMPLE.md
plan: ./PLAN_EXAMPLE.md#2-реалізувати-indicators
from: Agent B — Redis indicator
to: Integrator — Health module
status: Ready for Review
updated: 2026-08-05T20:30:00Z
---

# Handoff: Redis readiness indicator

## Мета

Передати готовий Redis indicator для readiness-перевірки зі сценаріями success, error і timeout.

## Scope

**Входило:** Redis indicator, його інтерфейс і unit-тести.

**Не входило:** HealthService, HTTP controller, PostgreSQL indicator, Compose та e2e.

## Змінено

| Файл/компонент                                           | Зміна                                                          |
| -------------------------------------------------------- | -------------------------------------------------------------- |
| `apps/api/src/health/indicators/redis.indicator.ts`      | Додано `check()` через наявний Redis client із timeout 1000 мс |
| `apps/api/src/health/indicators/redis.indicator.spec.ts` | Додано success, error і timeout тести                          |
| `apps/api/src/health/indicators/health-indicator.ts`     | Додано спільний контракт `HealthIndicator`                     |

## Прийняті рішення

| Рішення                          | Причина                                              | Джерело             |
| -------------------------------- | ---------------------------------------------------- | ------------------- |
| Не повертати текст Redis error   | Контракт не повинен розкривати внутрішні деталі      | SPEC-2026-001, AC-4 |
| Не створювати новий Redis client | Readiness має використовувати чинний connection pool | SPEC-2026-001, NFR  |
| Timeout обробляти як `down`      | Оркестратору потрібен однозначний результат          | SPEC-2026-001, FR-4 |

## Перевірено

| Команда або сценарій                       | Результат | Примітка  |
| ------------------------------------------ | --------- | --------- |
| `pnpm test -- redis.indicator.spec.ts`     | PASS      | 3/3 тести |
| `pnpm lint apps/api/src/health/indicators` | PASS      | 0 помилок |
| `pnpm typecheck`                           | PASS      | 0 помилок |

## Не завершено

- Інтеграція indicator у `HealthService`.
- E2E-перевірка відповіді `503`.

Ці пункти не входили в scope Agent B і залишаються за інтегратором.

## Відомі ризики

- Реальний Redis container ще не зупинявся вручну; unit-тест використовує контрольований fake client.

## Поточний стан

- Безпечний стан файлів: unit-тести indicator проходять незалежно від Docker.
- Відомі failing checks: немає.
- Незбережені/неінтегровані зміни: три файли зі списку вище, commit не створено.

## Наступна дія

1. Інтегратор перечитує diff і повторно запускає `pnpm test -- redis.indicator.spec.ts`.
2. Підключає `RedisIndicator` до `HealthService` разом із PostgreSQL indicator.
3. Запускає health e2e-тести та ручну Compose-перевірку.

## Питання, що потребують рішення

Немає.
