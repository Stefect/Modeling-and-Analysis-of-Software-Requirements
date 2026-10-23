# Sequence-діаграми за сценаріями (Завдання 3)

Деталізують `task-3/spec.md`, розд. 3. Одна діаграма на один сценарій Gherkin з `task-2/README.md`, розд. 3:
- **Діаграма 1 — US-01:** «Створення замовлення з Price Snapshot і резервуванням залишку» (UC_CreateOrder, `<<include>>` UC_Snapshot).
- **Діаграма 2 — US-02:** «Онлайн-оплата як extend, успіх і відмова з поверненням резерву» (UC_Pay, `<<extend>>`).

Еталон PlantUML з тими самими повідомленнями: `task-3/sequence.puml`.

## Діаграма 1. US-01: створення замовлення

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant WebApp
    participant UserService
    participant OrderService
    participant CatalogService
    participant Database

    Customer->>WebApp: confirmOrder(cartItems)
    WebApp->>UserService: validateUser(sessionToken) [EARS-14]
    UserService->>Database: findUser(userId)
    Database-->>UserService: role
    alt Сесія недійсна або прострочена [EARS-14]
        UserService-->>WebApp: 401 Unauthorized
        WebApp-->>Customer: loginRequired
    else Сесія дійсна, role = customer
        UserService-->>WebApp: userId, role
        WebApp->>OrderService: createOrder(userId, cartItems)
        activate OrderService
        OrderService->>CatalogService: getProducts(productIds)
        CatalogService->>Database: findProducts(productIds)
        Database-->>CatalogService: price, stock_quantity
        CatalogService-->>OrderService: products
        activate Database
        alt Недостатньо stock_quantity [EARS-03]
            OrderService-->>WebApp: outOfStock
            WebApp-->>Customer: outOfStock
        else stock_quantity достатньо
            OrderService->>Database: insertOrder(status = Pending) [EARS-02]
            OrderService->>Database: insertOrderItems(unit_price snapshot) [EARS-02, include UC_Snapshot]
            loop Для кожної позиції ORDER_ITEM [EARS-03]
                OrderService->>Database: reserveStock(productId, quantity)
            end
            OrderService-->>WebApp: orderId, status = Pending
            WebApp-->>Customer: orderCreated(orderId)
        end
        deactivate Database
        deactivate OrderService
    end
```

## Діаграма 2. US-02: онлайн-оплата

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant WebApp
    participant UserService
    participant OrderService
    participant Database
    participant PaymentGateway

    Customer->>WebApp: payOrder(orderId)
    WebApp->>UserService: validateUser(sessionToken) [EARS-14]
    UserService->>Database: findUser(userId)
    Database-->>UserService: role
    UserService-->>WebApp: userId, role
    WebApp->>OrderService: payOrder(orderId, userId)
    activate OrderService
    OrderService->>Database: findOrder(orderId)
    Database-->>OrderService: status = Pending, total_amount
    opt Онлайн-оплата обрана [EARS-04, extend UC_Pay]
        OrderService->>PaymentGateway: charge(orderId, total_amount)
        activate Database
        alt Оплата approved [EARS-04]
            PaymentGateway-->>OrderService: approved
            OrderService->>Database: updateOrder(status = Paid, payment_status = approved)
        else Оплата declined [EARS-05]
            PaymentGateway-->>OrderService: declined
            loop Для кожної позиції ORDER_ITEM [EARS-05]
                OrderService->>Database: releaseStock(productId, quantity)
            end
            OrderService->>Database: updateOrder(status = Unpaid)
        end
        deactivate Database
    end
    OrderService-->>WebApp: paymentResult
    deactivate OrderService
    WebApp-->>Customer: orderStatus
```

## Легенда
- `alt` / `else` — гілки: сесія (EARS-14), залишок (EARS-03), результат оплати (EARS-04, EARS-05).
- `opt` — онлайн-оплата необов'язкова. Якщо її не обрано, `Payment Gateway` не викликається, `ORDER` лишається `Pending` (сценарій «Оплата не обрана»).
- `loop` — резерв по кожній позиції (EARS-03) і повернення резерву при відмові (EARS-05).
- `include UC_Snapshot` — `unit_price` записується в тому самому кроці, що й `ORDER_ITEM`, поза `opt` (EARS-02).
- `activate` / `deactivate`:
  - Діаграма 1: `OrderService` і `Database` активні від початку `alt Недостатньо stock_quantity` до його кінця.
  - Діаграма 2: `OrderService` активний на всьому виконанні `payOrder`; `Database` активний від початку `alt` (approved/declined) до його кінця всередині `opt`.
- `par` не використано: у сценаріях US-01 і US-02 немає кроків, які виконуються паралельно. Кроки йдуть послідовно за логікою сесії, залишку та оплати.
- Пунктирна стрілка `-->>` — return-повідомлення.
- `PaymentGateway` відсутній на діаграмі 1: сценарій US-01 не містить оплати.
- Не показано: повторну оплату `Unpaid` (EARS-13), автоскасування (EARS-12), SQL і транзакції. Це окремі сценарії або деталі реалізації.
