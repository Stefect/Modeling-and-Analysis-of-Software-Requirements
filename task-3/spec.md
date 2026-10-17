# Специфікація взаємодії: оформлення замовлення (Завдання 3)

## 1. Намір і критерії
Описати динаміку ключового сценарію `UC_CreateOrder` (актор `Customer`) у вигляді sequence-діаграми: хто з ким взаємодіє, у якому порядку, з якими поверненнями і гілками (`alt`/`opt`/`loop`). Діаграма деталізує UC із Завдання 2 і має збігатися з ERD Завдання 1.

Критерії (фіксуються комітом):
1. Діаграма покриває основний потік `UC_CreateOrder`, включно з `<<include>>` `UC_Snapshot` і `<<extend>>` `UC_Pay`.
2. Учасники відповідають сутностям ERD (`USER`, `ORDER`, `ORDER_ITEM`, `PRODUCT`) або зовнішнім системам (`Payment Gateway`).
3. Кожне повідомлення-виклик має реалістичну назву і посилання на EARS (`EARS-02..05`, `EARS-14`).
4. Є щонайменше 3 гілки: помилка автентифікації, недостатній залишок, відмова оплати; є `opt` (онлайн-оплата) і `loop` (резерв по позиціях).
5. Mermaid-блок відображається на GitHub; еталонна UML-версія — `sequence.puml` (PlantUML).

## 2. Учасники
| Учасник | Тип | Відповідність моделі |
| :--- | :--- | :--- |
| `Customer` | Актор | Нащадок `User` (Завдання 2) |
| `WebApp` | Компонент UI | Клієнтський застосунок (поза ERD) |
| `OrderService` | Сервіс | Логіка `UC_CreateOrder`, `UC_Snapshot` |
| `UserService` | Сервіс | Перевірка сесії і ролі за `USER` |
| `CatalogService` | Сервіс | Читання `PRODUCT` (у цьому сценарії `CATEGORY` не читається) |
| `Database` | Сховище | Таблиці `USER`, `PRODUCT`, `ORDER`, `ORDER_ITEM` |
| `PaymentGateway` | Зовнішня система | `Payment Gateway` (Завдання 2) |

## 3. Ключові кроки сценарію
1. `Customer` → `WebApp`: відкриває кошик і підтверджує замовлення.
2. `WebApp` → `UserService`: `validateUser(sessionToken)` → `USER.role = customer`. Гілка `alt`: сесія недійсна — відмова 401, без створення `ORDER` (EARS-14).
3. `OrderService` → `CatalogService`: `getProductList(productIds)` — актуальні `PRODUCT.price` і `stock_quantity`.
4. `alt`: якщо `stock_quantity < ORDER_ITEM.quantity` — відмова, `ORDER` не створюється.
5. `OrderService` → `Database`: створює `ORDER` (`status = Pending`) і `ORDER_ITEM` з `unit_price` (EARS-02, `<<include>>` UC_Snapshot).
6. `loop` по кожній позиції: `PRODUCT.stock_quantity -= ORDER_ITEM.quantity` (EARS-03).
7. `opt`: онлайн-оплата — `PaymentGateway.charge(orderId, total)` (EARS-04, `<<extend>>` UC_Pay).
   - `alt`: успіх → `ORDER.status = Paid`, `ORDER.payment_status = approved` (`task-2/README.md`, US-02).
   - `alt`: відмова → окремо повернення резерву, потім `ORDER.status = Unpaid` (EARS-05).
8. `OrderService` → `WebApp`: повертає номер замовлення і статус.

## 4. Межі
- Автоскасування через 24 год (EARS-12) у діаграму не входить: це окремий сценарій `System Timer`, його можна додати пізніше.
- Оплата не є обов'язковою: гілка `opt` позначає, що замовлення створюється і без неї.
- Ім'я `getBookList()` із методички — приклад з іншої предметної області. Тут відповідний метод — `getProductList()`, зміну зафіксовано як розбіжність (див. `DEFENSE.md`, після рев'ю).

## 5. Відомі ризики
- Стан `Unpaid` при відмові оплати: за EARS-05 повторна оплата — EARS-13; на діаграмі не показано.
- Атомарність резерву (транзакція БД) — деталь реалізації, не моделі.
