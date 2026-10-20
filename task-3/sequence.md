# Sequence-діаграма: оформлення замовлення (UC_CreateOrder)

> **ТИМЧАСОВО:** блоки `alt`, `opt`, `loop` замінено на нотатки «×». Повна версія з гілками — у коміті `416d605`.

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
    Note over UserService,WebApp: × Сесія недійсна: помилка 401, потрібно увійти
    UserService-->>WebApp: userId, role
    WebApp->>OrderService: submitOrder(userId, cartItems)
    OrderService->>CatalogService: getProductList(productIds)
    CatalogService->>Database: Читання PRODUCT
    Database-->>CatalogService: price, stock_quantity
    CatalogService-->>OrderService: Товари з ціною і залишком
    Note over OrderService,WebApp: × Недостатньо залишку: відмова, ORDER не створюється
    OrderService->>Database: Створити ORDER (status = Pending)
    OrderService->>Database: Створити ORDER_ITEM з unit_price
    Note over OrderService,Database: × Для кожної позиції ORDER_ITEM
    OrderService->>Database: Зменшити PRODUCT.stock_quantity
    Note over OrderService,PaymentGateway: × Онлайн-оплата (UC_Pay, необов'язкова)
    OrderService->>PaymentGateway: Списати total_amount
    PaymentGateway-->>OrderService: approved
    OrderService->>Database: ORDER.status = Paid, payment_status = approved
    Note over OrderService,PaymentGateway: × Оплата відхилена
    PaymentGateway-->>OrderService: declined
    OrderService->>Database: Повернути залишок (EARS-05)
    OrderService->>Database: ORDER.status = Unpaid
    OrderService-->>WebApp: orderId, статус
    WebApp-->>Customer: Замовлення створено
```

## Легенда
- «×» — місце, де в повній версії були `alt` (умовні гілки), `opt` (необов'язкова оплата) або `loop` (резерв по позиціях).
- Еталон PlantUML: `task-3/sequence.puml` (також у тимчасовій версії).
