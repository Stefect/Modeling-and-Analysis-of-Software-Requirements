# Component Diagram: архітектура маркетплейсу (Завдання 5)

Діаграма подана як **оглядова** компонентна модель системи (спрощено для читабельності). Детальні provided/required-інтерфейси для кожного зв'язку наведено в таблиці `spec.md` §4. Еталонна повна UML-версія з усіма інтерфейсами (льодяники/розетки) — у `task-5/component.puml`.

## Компоненти та залежності (огляд)

```mermaid
flowchart LR
    %% --- Клієнт ---
    subgraph Client["Клієнт"]
        WebClient["Web Client (SPA)"]
    end

    %% --- API ---
    subgraph API["API"]
        Gateway["API Gateway / BFF"]
    end

    %% --- Сервіси ---
    subgraph Services["Сервіси"]
        AuthService["Auth Service"]
        CatalogService["Catalog Service"]
        OrderService["Order Service"]
        SellerReviewService["Seller and Review Service"]
        NotificationService["Notification Service"]
    end

    %% --- Інфраструктура ---
    subgraph Infra["Інфраструктура"]
        PaymentAdapter["Payment Adapter"]
        OutboxRelay["Outbox Relay"]
        ExpiryScheduler["Expiry Scheduler"]
        DataAccess["Data Access"]
    end

    %% --- Дані ---
    subgraph Data["Дані"]
        Database[("PostgreSQL")]
    end

    %% --- Зовнішні ---
    subgraph External["Зовнішні системи"]
        PaymentGateway["Payment Gateway"]
        EmailProvider["Email and SMS"]
        IdentityProvider["Identity Provider (опц.)"]
    end

    %% --- Зв'язки ---
    WebClient --> Gateway
    Gateway --> AuthService
    Gateway --> CatalogService
    Gateway --> OrderService
    Gateway --> SellerReviewService
    Gateway --> NotificationService

    AuthService --> DataAccess
    CatalogService --> DataAccess
    OrderService --> DataAccess
    SellerReviewService --> DataAccess

    OrderService --> PaymentAdapter
    OrderService --> OutboxRelay
    OutboxRelay --> DataAccess
    OutboxRelay --> NotificationService
    ExpiryScheduler --> OrderService

    PaymentAdapter --> PaymentGateway
    NotificationService --> EmailProvider
    AuthService -.-> IdentityProvider

    DataAccess --> Database
```

## Примітки

* `Order Service` реалізує бізнес-логіку Завдання 4: резерв, перехід `Pending → Paid/Unpaid`, запис `OrderPaidEvent` в `outbox`, повтори (1 + 3) і DLQ.
* `Outbox Relay` зчитує `outbox` після коміту і ініціює `Notification Service`; збій відправки не відкочує оплату.
* `Expiry Scheduler` реалізує EARS-12: `Pending` понад 24 год → `Cancelled` + повернення залишків.
* `Identity Provider` позначено пунктиром як опційна інтеграція (автентифікація за `USER.password_hash`, EARS-11).
* Детальні інтерфейси (`IOrderService`, `IOrderRepository`, `IPaymentGateway` тощо) — у `spec.md` §4.