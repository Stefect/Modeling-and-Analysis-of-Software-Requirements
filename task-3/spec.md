# Специфікація взаємодії: оформлення замовлення (Завдання 3)

## 1. Намір і критерії
Описати динаміку ключового сценарію `UC_CreateOrder` (актор `Customer`) у вигляді sequence-діаграми: хто з ким взаємодіє, у якому порядку, з якими поверненнями і гілками (`alt`/`opt`/`loop`). Діаграма деталізує UC із Завдання 2 і має збігатися з ERD Завдання 1.

Критерії (фіксуються комітом):
1. Дві діаграми за сценаріями Gherkin: US-01 покриває `UC_CreateOrder` з `<<include>>` `UC_Snapshot` (поза `opt`); US-02 покриває `UC_Pay` як `<<extend>>` (всередині `opt`).
2. Кожен учасник є актором, компонентом, що читає або пише сутності ERD (`USER`, `PRODUCT`, `ORDER`, `ORDER_ITEM`), або зовнішньою системою (`Payment Gateway`). Компоненти дозволені методичкою (завдання: «основні компоненти/об'єкти»).
3. Кожне повідомлення-виклик має реалістичну назву і посилання на EARS (`EARS-02..05`, `EARS-14`); return-повідомлення позначено пунктирною стрілкою.
4. Є щонайменше 3 гілки: помилка автентифікації і недостатній залишок (US-01), відмова оплати (US-02); є `opt` (онлайн-оплата, US-02) і `loop` (резерв по позиціях, US-01 і US-02).
5. Mermaid-блоки відображаються на GitHub; еталонна UML-версія — `sequence.puml` (PlantUML). Обидві версії містять однаковий потік у кожній діаграмі.
6. Статуси і залишки відповідають EARS: після успішної оплати `ORDER.status = Paid`, `payment_status = approved` (US-02); при відмові оплати — два окремі повідомлення: повернення резерву (EARS-05) і `ORDER.status = Unpaid`.

## 2. Учасники
| Учасник | Тип | Відповідність моделі |
| :--- | :--- | :--- |
| `Customer` | Актор | Нащадок `User` (Завдання 2) |
| `WebApp` | Компонент UI | Клієнтський застосунок (поза ERD) |
| `OrderService` | Сервіс | Логіка `UC_CreateOrder`, `UC_Snapshot` |
| `UserService` | Сервіс | Перевірка сесії і ролі, читає `USER` |
| `CatalogService` | Сервіс | Читання `PRODUCT` (у цьому сценарії `CATEGORY` не читається) |
| `Database` | Сховище | Таблиці `USER`, `PRODUCT`, `ORDER`, `ORDER_ITEM` |
| `PaymentGateway` | Зовнішня система | `Payment Gateway` (Завдання 2) |

## 3. Ключові кроки сценаріїв
Кроки 1–6 і 8 — US-01 (діаграма 1). Крок 7 — US-02 (діаграма 2).
1. `Customer` → `WebApp`: відкриває кошик і підтверджує замовлення.
2. `WebApp` → `UserService`: `validateUser(sessionToken)` [EARS-14]. `UserService` → `Database`: читання `USER.id`, `USER.role`. Гілка `alt`: сесія недійсна або прострочена — відмова 401, без створення `ORDER`.
3. `OrderService` → `CatalogService`: `getProductList(productIds)` — актуальні `PRODUCT.price` і `stock_quantity`.
4. `alt`: якщо `stock_quantity < ORDER_ITEM.quantity` — відмова, `ORDER` не створюється [EARS-03].
5. `OrderService` → `Database`: створює `ORDER` (`status = Pending`) і `ORDER_ITEM` з `unit_price` [EARS-02, `<<include>>` UC_Snapshot].
6. `loop` по кожній позиції: `PRODUCT.stock_quantity -= ORDER_ITEM.quantity` [EARS-03].
7. `opt`: онлайн-оплата — `PaymentGateway.charge(orderId, total_amount)` [EARS-04, `<<extend>>` UC_Pay].
   - `alt`: успіх → `ORDER.status = Paid`, `ORDER.payment_status = approved` (`task-2/README.md`, US-02).
   - `alt`: відмова → окремо повернення резерву, потім `ORDER.status = Unpaid` [EARS-05].
8. `OrderService` → `WebApp`: повертає номер замовлення і статус.

## 4. Межі
- Автоскасування через 24 год (EARS-12) у діаграму не входить: це окремий сценарій `System Timer`, його можна додати пізніше.
- Оплата не є обов'язковою: гілка `opt` позначає, що замовлення створюється і без неї (статус `Pending`).
- Ім'я `getBookList()` із методички — приклад з іншої предметної області. Тут відповідний метод — `getProductList()`, зміну зафіксовано в `docs/adr/0011-sequence-participants-split.md`.

## 5. Відомі ризики
- Повторна оплата `Unpaid` — EARS-13; на діаграмі не показано (окремий сценарій).
- Атомарність резерву (транзакція БД) — деталь реалізації, не моделі.
