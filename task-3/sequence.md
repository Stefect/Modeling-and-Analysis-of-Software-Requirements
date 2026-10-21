# Sequence-діаграма: оформлення замовлення (UC_CreateOrder)

Деталізує `task-3/spec.md`, розд. 3. Учасники — розд. 2; вимоги — EARS-02..05, EARS-14 (`task-2/README.md`). Еталон PlantUML: `task-3/sequence.puml`.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant WebApp
    participant UserService
    participant OrderService
    participant CatalogService
    participant Database
    participant PaymentGateway

    Customer->>WebApp: Підтверджує замовлення (UC_CreateOrder)
    WebApp->>UserService: validateUser(sessionToken) [EARS-14]
    UserService->>Database: Читання USER (id, role)
    Database-->>UserService: USER.role
    alt Сесія недійсна або прострочена
        UserService-->>WebApp: Помилка 401 [EARS-14]
        WebApp-->>Customer: Потрібно увійти
    else Сесія дійсна (role = customer)
        UserService-->>WebApp: userId, role
        WebApp->>OrderService: submitOrder(userId, cartItems)
        OrderService->>CatalogService: getProductList(productIds)
        CatalogService->>Database: Читання PRODUCT (price, stock_quantity)
        Database-->>CatalogService: price, stock_quantity
        CatalogService-->>OrderService: Товари з ціною і залишком
        alt Недостатньо залишку [EARS-03]
            OrderService-->>WebApp: Відмова, немає на складі
            WebApp-->>Customer: Товар недоступний
        else Залишку достатньо
            OrderService->>Database: Створити ORDER (status = Pending) [EARS-02]
            OrderService->>Database: Створити ORDER_ITEM з unit_price (include UC_Snapshot) [EARS-02]
            loop Для кожної позиції ORDER_ITEM [EARS-03]
                OrderService->>Database: Зменшити PRODUCT.stock_quantity на ORDER_ITEM.quantity
            end
            opt Онлайн-оплата (extend UC_Pay) [EARS-04]
                OrderService->>PaymentGateway: charge(orderId, total_amount)
                alt Оплата успішна
                    PaymentGateway-->>OrderService: approved
                    OrderService->>Database: ORDER.status = Paid, payment_status = approved
                else Оплата відхилена [EARS-05]
                    PaymentGateway-->>OrderService: declined
                    OrderService->>Database: Повернути PRODUCT.stock_quantity (скасування резерву)
                    OrderService->>Database: ORDER.status = Unpaid
                end
            end
            OrderService-->>WebApp: orderId, статус
            WebApp-->>Customer: Замовлення створено
        end
    end
```

## Легенда
- `alt` / `else` — умовні гілки: сесія (EARS-14), залишок, результат оплати (EARS-05).
- `opt` — онлайн-оплата необов'язкова (`<<extend>>` UC_Pay, EARS-04).
- `loop` — резерв по кожній позиції (EARS-03).
- `include UC_Snapshot` — знімок `unit_price` створюється в тому самому кроці, що й `ORDER_ITEM` (EARS-02). Включений UC виконується завжди, тому стоїть поза `opt`.
- Пунктирна стрілка `-->>` — return-повідомлення.
- Назви дій, а не внутрішні деталі: SQL і транзакції не показано (рівень моделі, а не реалізації).
