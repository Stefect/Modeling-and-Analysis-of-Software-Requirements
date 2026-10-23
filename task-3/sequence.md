# Sequence-діаграми за сценаріями (Завдання 3)

Деталізують `task-3/spec.md`. Одна діаграма на один Gherkin-сценарій з `task-2/README.md`, розд. 3. Учасники і сигнатури методів — за `docs/adr/0011-sequence-participants-split.md`. Розбиття на окремі діаграми — за `docs/adr/0013-sequence-four-scenarios.md`.

Статус: 4 з 4.

| № | Сценарій (Gherkin, Завдання 2) | Вимоги | Комбінований фрагмент |
|---|---|---|---|
| S1 | US-01: «Створення замовлення з Price Snapshot і резервуванням залишку» | EARS-02, EARS-03, EARS-14 | `critical`, `loop` |
| S2 | US-02: «Повторна оплата відхиляється, якщо товару немає на складі» | EARS-13, EARS-14 | `break` |
| S3 | US-02: «Відмова платежу скасовує резерв» | EARS-04, EARS-05, EARS-14 | `critical`, `loop` |
| S4 | US-02: «Оплата не обрана (extend не виконується)» | EARS-04, EARS-14 | `opt` |

---

## Діаграма 1. S1: успішне оформлення замовлення

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant WebApp
    participant UserService
    participant OrderService
    participant CatalogService
    participant Database

    Customer->>+WebApp: confirmOrder(sessionToken, cartItems)
    WebApp->>+UserService: validateSession(sessionToken)
    UserService->>+Database: getSession(sessionToken)
    Database-->>-UserService: userId
    UserService->>+Database: getUserRole(userId)
    Database-->>-UserService: role = 'customer'
    UserService-->>-WebApp: userId, role = 'customer'
    WebApp->>+OrderService: createOrder(userId, cartItems)
    OrderService->>+CatalogService: getProductList(productIds)
    CatalogService->>+Database: findProducts(productIds)
    Database-->>-CatalogService: products(price, stock_quantity)
    CatalogService-->>-OrderService: products(price, stock_quantity)
    critical Транзакція створення замовлення
        OrderService->>+Database: insertOrder(userId, totalAmount, 'Pending')
        Database-->>-OrderService: orderId
        OrderService->>+Database: insertOrderItems(orderId, items, unit_price)
        Database-->>-OrderService: ok
        loop для кожного ORDER_ITEM
            OrderService->>+Database: reserveStock(productId, qty)
            Database-->>-OrderService: reserved
        end
    end
    OrderService-->>-WebApp: orderId, status = 'Pending'
    WebApp-->>-Customer: orderId, status = 'Pending'
```

**Легенда.** `critical` об'єднує створення `ORDER` і `ORDER_ITEM` з резервуванням в одну транзакцію. `loop` резервує кожну позицію (EARS-03). `unit_price` записується тим самим `insertOrderItems` (EARS-02, include UC_Snapshot).

---

## Діаграма 2. S2: нестача залишку при повторній оплаті

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant WebApp
    participant UserService
    participant OrderService
    participant CatalogService
    participant Database

    Customer->>+WebApp: payOrder(sessionToken, orderId)
    WebApp->>+UserService: validateSession(sessionToken)
    UserService->>+Database: getSession(sessionToken)
    Database-->>-UserService: userId
    UserService->>+Database: getUserRole(userId)
    Database-->>-UserService: role = 'customer'
    UserService-->>-WebApp: userId, role = 'customer'
    WebApp->>+OrderService: retryPayment(orderId, userId)
    OrderService->>+Database: findOrder(orderId)
    Database-->>-OrderService: status = 'Unpaid', items(productId, quantity)
    OrderService->>+CatalogService: getProductList(productIds)
    CatalogService->>+Database: findProducts(productIds)
    Database-->>-CatalogService: products(stock_quantity)
    CatalogService-->>-OrderService: products(stock_quantity)
    break Недостатньо stock_quantity для ORDER_ITEM
        OrderService-->>-WebApp: insufficientStock(productId)
        WebApp-->>-Customer: retryRejected, status = 'Unpaid'
    end
```

**Легенда.** `break` завершує сценарій, якщо `stock_quantity < ORDER_ITEM.quantity` (EARS-13). Запису в `Database` немає, `ORDER` лишається `Unpaid`, `PaymentGateway` не викликається.

---

## Діаграма 3. S3: відмова онлайн-оплати і скасування резерву

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant WebApp
    participant UserService
    participant OrderService
    participant PaymentGateway
    participant Database

    Customer->>+WebApp: payOrder(sessionToken, orderId)
    WebApp->>+UserService: validateSession(sessionToken)
    UserService->>+Database: getSession(sessionToken)
    Database-->>-UserService: userId
    UserService->>+Database: getUserRole(userId)
    Database-->>-UserService: role = 'customer'
    UserService-->>-WebApp: userId, role = 'customer'
    WebApp->>+OrderService: payOrder(orderId, userId)
    OrderService->>+Database: findOrder(orderId)
    Database-->>-OrderService: status = 'Pending', total_amount, items(productId, quantity)
    OrderService->>+PaymentGateway: charge(orderId, total_amount)
    PaymentGateway-->>-OrderService: declined
    critical Транзакція скасування резерву
        loop для кожного ORDER_ITEM
            OrderService->>+Database: releaseStock(productId, quantity)
            Database-->>-OrderService: released
        end
        OrderService->>+Database: updateOrder(orderId, status = 'Unpaid', payment_status = 'declined')
        Database-->>-OrderService: ok
    end
    OrderService-->>-WebApp: paymentResult = 'declined'
    WebApp-->>-Customer: paymentResult = 'declined', status = 'Unpaid'
```

**Легенда.** `critical` містить повернення резерву і зміну статусу, щоб не було частково скасованого замовлення (EARS-05). `releaseStock` повертає `ORDER_ITEM.quantity` у `PRODUCT.stock_quantity`.

---

## Діаграма 4. S4: оформлення без онлайн-оплати

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant WebApp
    participant UserService
    participant OrderService
    participant PaymentGateway
    participant Database

    Customer->>+WebApp: continueWithoutPayment(sessionToken, orderId)
    WebApp->>+UserService: validateSession(sessionToken)
    UserService->>+Database: getSession(sessionToken)
    Database-->>-UserService: userId
    UserService->>+Database: getUserRole(userId)
    Database-->>-UserService: role = 'customer'
    UserService-->>-WebApp: userId, role = 'customer'
    WebApp->>+OrderService: confirmWithoutPayment(orderId, userId)
    OrderService->>+Database: findOrder(orderId)
    Database-->>-OrderService: status = 'Pending', total_amount
    opt Онлайн-оплата обрана (у цьому сценарії не виконується)
        OrderService->>+PaymentGateway: charge(orderId, total_amount)
        PaymentGateway-->>-OrderService: approved / declined
    end
    OrderService-->>-WebApp: orderId, status = 'Pending'
    WebApp-->>-Customer: orderId, status = 'Pending'
```

**Легенда.** `opt` показує, що `PaymentGateway` викликається тільки за вибору онлайн-оплати. У цьому сценарії гілка не виконується, `ORDER` лишається `Pending` без змін у `Database`.

---

## Загальні правила

- Кожен виклик має пару `->>+` / `-->>-`. Учасник активний від свого першого виклику до повернення результату.
- Перевірка ролі йде через `Database`: `getSession` і `getUserRole`. Роль не береться без звернення до БД.
- Прозових назв немає. Усі повідомлення є сигнатурами методів.
- Не показано: SQL, індекси, деталі транзакцій поза `critical`, SMS/email.
