# Специфікація взаємодії: оформлення замовлення (Завдання 3)

## 1. Намір і критерії
Описати динаміку ключового сценарію `UC_CreateOrder` (актор `Customer`) у вигляді sequence-діаграм: хто з ким взаємодіє, у якому порядку, з якими поверненнями і гілками (`alt`/`opt`/`loop`/`break`). Діаграми деталізують UC із Завдання 2 і мають збігатися з ERD Завдання 1.

Критерії (фіксуються комітом):
1. Вісім діаграм S1–S8, по одній на кожен Gherkin-сценарій US-01 і US-02 з `task-2/README.md`, розд. 3, крім автоскасування через 24 год (`UC_ExpireOrder`, поза межами, розд. 4). US-01 покриває `UC_CreateOrder` з `<<include>>` `UC_Snapshot` (поза `opt`) — S1, S6, S7; US-02 покриває `UC_Pay` як `<<extend>>` (всередині `opt`) і повторну оплату — S2–S5, S8.
2. Кожен учасник є актором, компонентом, що читає або пише сутності ERD (`USER`, `PRODUCT`, `ORDER`, `ORDER_ITEM`), або зовнішньою системою (`Payment Gateway`). Компоненти дозволені методичкою (завдання: «основні компоненти/об'єкти»).
3. Кожне повідомлення-виклик має реалістичну назву і посилання на EARS (`EARS-02..05`, `EARS-13`, `EARS-14`); return-повідомлення позначено пунктирною стрілкою.
4. Є щонайменше 3 гілки: помилка автентифікації і недостатній залишок (US-01, S1), відмова оплати (US-02, S3, S5); є `opt` (онлайн-оплата, S4, S5), `loop` (резерв по позиціях, S1, S3, S5) і `break` (повторна оплата і нестача при оформленні, S2, S6).
5. Mermaid-блоки відображаються на GitHub; еталонна UML-версія — `sequence.puml` (PlantUML). Обидві версії містять однаковий потік у кожній діаграмі.
6. Статуси і залишки відповідають EARS: після успішної оплати `ORDER.status = Paid`, `payment_status = approved` (US-02, S5); при відмові оплати — два окремі повідомлення: повернення резерву (EARS-05) і `ORDER.status = Unpaid` (S3, S5).

## 2. Учасники
| Учасник | Тип | Відповідність моделі |
| :--- | :--- | :--- |
| `Customer` | Актор | Нащадок `User` (Завдання 2) |
| `WebApp` | Компонент UI | Клієнтський застосунок (поза ERD) |
| `OrderService` | Сервіс | Логіка `UC_CreateOrder`, `UC_Snapshot`, `UC_Pay` |
| `UserService` | Сервіс | Перевірка сесії і ролі, читає `USER` |
| `CatalogService` | Сервіс | Читання `PRODUCT` (у цьому сценарії `CATEGORY` не читається) |
| `Database` | Сховище | Таблиці `USER`, `PRODUCT`, `ORDER`, `ORDER_ITEM` |
| `PaymentGateway` | Зовнішня система | `Payment Gateway` (Завдання 2) |

## 3. Ключові кроки сценаріїв
Діаграми за сценаріями (`sequence.md`):
- S1 — US-01: створення замовлення з резервом (кроки 1–6, 8).
- S2 — US-02: повторна оплата відхиляється через нестачу (`break`, EARS-13).
- S3 — US-02: відмова онлайн-оплати і скасування резерву (кроки 7b, 8).
- S4 — US-02: оплата не обрана (`opt` не виконується).
- S5 — US-02: успішна онлайн-оплата (`opt` → `alt`, крок 7a).
- S6 — US-01: нестача залишку при оформленні (`break`, EARS-03, варіант 3 з ADR 0013).
- S7 — US-01: зміна ціни після оформлення не змінює `ORDER_ITEM.unit_price` (`alt` ролі продавця, EARS-02, EARS-09).
- S8 — US-02: повторна оплата після відмови з повторним резервом (`critical`, `loop`, `alt`, EARS-13).

Основний потік S1 і спільні кроки:
1. `Customer` → `WebApp`: відкриває кошик і підтверджує замовлення, `confirmOrder(sessionToken, cartItems)`.
2. `WebApp` → `UserService`: `validateSession(sessionToken)` [EARS-14]. `UserService` → `Database`: `getSession(sessionToken)` і `getUserRole(userId)`, читання `USER.id`, `USER.role`. Гілка `alt`: сесія недійсна або прострочена — відмова 401, `ORDER` не створюється. Роль читається до `alt`, але для недійсної сесії не повертається.
3. `WebApp` → `OrderService`: `createOrder(userId, cartItems)`. `OrderService` → `CatalogService`: `getProductList(productIds)` — актуальні `PRODUCT.price` і `stock_quantity`.
4. `alt`: якщо `stock_quantity < ORDER_ITEM.quantity` — відмова `insufficientStock(productId)`, `ORDER` не створюється [EARS-03]. Інакше — кроки 5–6.
5. `OrderService` → `Database`: `insertOrder` і `insertOrderItems` з `unit_price` в одній транзакції (`critical`) [EARS-02, `<<include>>` UC_Snapshot]. `ORDER.status = Pending`.
6. `loop` по кожній позиції: `reserveStock(productId, qty)` — `PRODUCT.stock_quantity -= ORDER_ITEM.quantity` [EARS-03].
7. `opt`: онлайн-оплата — `charge(orderId, total_amount)` через `PaymentGateway` [EARS-04, `<<extend>>` UC_Pay].
   - a. `alt`, успіх (S5): `ORDER.status = Paid`, `ORDER.payment_status = approved` (`task-2/README.md`, US-02).
   - b. `alt`, відмова (S3, S5): у `critical` — `releaseStock(productId, qty)` для кожної позиції, потім `ORDER.status = Unpaid`, `payment_status = declined` [EARS-05]. Два окремі повідомлення.
8. `OrderService` → `WebApp` → `Customer`: повертає номер замовлення і статус.

## 4. Межі
- Автоскасування через 24 год (EARS-12) у діаграми не входить: це окремий сценарій `System Timer`, його можна додати пізніше.
- Оплата не є обов'язковою: гілка `opt` позначає, що замовлення створюється і без неї (статус `Pending`, S4).
- Повторна оплата з успішним резервом (`Unpaid` → `Pending`, US-02) — S8. Відхилення через нестачу при повторній оплаті — S2 (EARS-13).
- Зміна `PRODUCT.price` продавцем — S7 лише як вплив на вже створене замовлення; сама процедура редагування товару (`UC_ManageProduct`) не змодельована.
- Ім'я `getBookList()` із методички — приклад з іншої предметної області. Тут відповідний метод — `getProductList()`, зміну зафіксовано в `docs/adr/0011-sequence-participants-split.md`.

## 5. Відомі ризики
- Mermaid веде activation bars лінійно і не розрізняє гілки `alt`. Закриття активацій виконано через `deactivate` після фрагмента (S1, S2, S6, S7, S8), а повернення в гілках пунктирні (див. `sequence.md`, розд. «Загальні правила»). Це відхилення від початкового прикладу з ADR 0013.
- Атомарність резерву (транзакція БД, NFR-07) — деталь реалізації, не моделі.
