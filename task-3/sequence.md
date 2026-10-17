# Sequence-діаграма: оформлення замовлення (UC_CreateOrder)

Деталізує `task-3/spec.md`, розд. 3. Учасники — розд. 2; вимоги — EARS-02..05, EARS-14 (`task-2/README.md`).

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

    Customer->>WebApp: Підтверджує замовлення
    WebApp->>UserService: validateUser(sessionToken)
    alt Сесія недійсна
        UserService-->>WebApp: Помилка 401
        WebApp-->>Customer: Потрібно увійти
    else Сесія дійсна (role = customer)
        UserService-->>WebApp: userId, role
        WebApp->>OrderService: submitOrder(userId, cartItems)
        OrderService->>CatalogService: getProductList(productIds)
        CatalogService->>Database: Читання PRODUCT
        Database-->>CatalogService: price, stock_quantity
        CatalogService-->>OrderService: Товари з ціною і залишком
        alt Недостатньо залишку
            OrderService-->>WebApp: Відмова, немає на складі
            WebApp-->>Customer: Товар недоступний
        else Залишку достатньо
            OrderService->>Database: Створити ORDER (status = Pending)
            OrderService->>Database: Створити ORDER_ITEM з unit_price
            loop Для кожної позиції ORDER_ITEM
                OrderService->>Database: Зменшити PRODUCT.stock_quantity
            end
            opt Онлайн-оплата (UC_Pay, extend)
                OrderService->>PaymentGateway: Списати total_amount
                alt Оплата успішна
                    PaymentGateway-->>OrderService: approved
                    OrderService->>Database: ORDER.status = Paid, payment_status = approved
                else Оплата відхилена
                    PaymentGateway-->>OrderService: declined
                    OrderService->>Database: Повернути залишок (EARS-05)
                    OrderService->>Database: ORDER.status = Unpaid
                end
            end
            OrderService-->>WebApp: orderId, статус
            WebApp-->>Customer: Замовлення створено
        end
    end
```

## Легенда
- `alt` / `else` — умовні гілки (помилка сесії, нестача залишку, результат оплати).
- `opt` — онлайн-оплата необов'язкова (`<<extend>>` UC_Pay).
- `loop` — резерв по кожній позиції (EARS-03).
- Знімок ціни `unit_price` створюється в тому самому кроці, що й `ORDER_ITEM` (`<<include>>` UC_Snapshot, EARS-02).
- Пишуться назви дій, а не внутрішні деталі: SQL і транзакції не показано, це рівень моделювання, а не реалізації.
- Еталон PlantUML: `task-3/sequence.puml`.
