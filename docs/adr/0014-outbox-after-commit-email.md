# 14. Надсилання email після оплати через Transactional Outbox

* Статус: прийнято
* Дата: 2026-10-27
* Контекст: Завдання 4 (`task-4/`), `task-4/spec.md`, `task-4/activity.md`

## Context and Problem Statement

Після успішної оплати покупцю треба надіслати email-підтвердження (EARS-15). Надсилання не має випереджати коміт запису `ORDER.status = Paid` у БД і не повинно блокувати або відкочувати транзакцію оплати.

## Considered Options

* **Варіант 1:** email одразу після `Paid` без outbox.
* **Варіант 2:** fork без транзакції (статус і email паралельно).
* **Варіант 3:** Transactional Outbox: статус і подія в одній транзакції, відправка через фоновий Relay після коміту. **Обрано.**

## Decision Outcome

Обрано **варіант 3**. `ORDER.status = Paid` і `insertOutbox(OrderPaidEvent)` виконуються в одній транзакції. Після коміту фоновий Outbox Relay у складі `OrderService` надсилає email покупцю.

### Consequences

* Good: email не може випередити запис у БД; статус і подія узгоджені атомарно.
* Good: помилка email не відкочує оплату.
* Bad: потрібна таблиця `outbox` і фоновий воркер Relay.
* Bad: email приходить із затримкою (асинхронно).
* Bad: політика повторів (4 спроби, DLQ) потребує моніторингу DLQ.

## Links

* `task-4/spec.md` (EARS-15, рішення 2, 4, 7)
* `task-4/activity.md`
* `task-3/sequence.md`, діаграма S5 (розбіжність, рішення 7 у spec)
* `docs/adr/0013-sequence-four-scenarios.md`
