# Component Diagram: архітектура маркетплейсу (Завдання 5)

Діаграма подана як компонентна модель системи. Оскільки Mermaid не має окремої нотації UML Component Diagram, використано `flowchart` з пакетами (`subgraph`) і компонентами; стрілки підписуються іменами provided/required-інтерфейсів. Еталонна UML-версія з льодяниками/розетками інтерфейсів — у `task-5/component.puml`.

## Компоненти та залежності

```mermaid
flowchart LR
    %% --- Клієнт ---
    subgraph Client["Клієнтський рівень"]
        WebClient["Web Client (SPA)"]
    end

    %% --- API ---
    subgraph API["API-рівень"]
        Gateway["API Gateway / BFF"]
    end

    %% --- Сервіси ---
    subgraph Services["Прикладні сервіси"]
        AuthService["Auth Service"]
        CatalogService["Catalog Service"]
        SellerService["Seller Service"]
        OrderService["Order Service"]
        ReviewService["Review Service"]
        NotificationService["Notification Service"]
    end

    %% --- Інфраструктура ---
    subgraph Infra["Інфраструктура"]
        PaymentAdapter["Payment Adapter"]
        OutboxRelay["Outbox Relay Worker"]
        ExpiryScheduler["Order Expiry Scheduler"]
        DataAccess["Data Access (Repository)"]
    end

    %% --- Дані ---
    subgraph Data["Рівень даних"]
        Database[("Database (PostgreSQL)")]
    end

    %% --- Зовнішні системи ---
    subgraph External["Зовнішні системи"]
        PaymentGateway["Payment Gateway"]
        EmailProvider["Email/SMS Provider"]
        IdentityProvider["Identity Provider (OAuth/OIDC, опц.)"]
    end

    %% --- Зв'язки клієнт- API ---
    WebClient -->|IAuthApi ICatalogApi IOrderApi IReviewApi ISellerApi| Gateway

    %% --- API -> сервіси ---
    Gateway -->|IAuthService| AuthService
    Gateway -->|ICatalogService| CatalogService
    Gateway -->|IOrderService| OrderService
    Gateway -->|IReviewService| ReviewService
    Gateway -->|ISellerService| SellerService

    %% --- Сервіси -> інфраструктура/дані ---
    AuthService -->|IUserRepository| DataAccess
    CatalogService -->|IProductRepository ICategoryRepository IStoreRepository| DataAccess
    SellerService -->|IStoreRepository, IProductRepository| DataAccess
    OrderService -->|IOrderRepository IProductRepository IOutboxRepository| DataAccess
    ReviewService -->|IReviewRepository| DataAccess
    OrderService -->|IPaymentGateway| PaymentAdapter
    OrderService -->|IOutboxRepository події| OutboxRelay

    %% --- Фонові процеси ---
    OutboxRelay -->|IOutboxRepository| DataAccess
    OutboxRelay -->|IEmailSender| NotificationService
    ExpiryScheduler -->|IOrderService expire| OrderService

    %% --- Зовнішні інтеграції ---
    PaymentAdapter -->|зовнішній API| PaymentGateway
    NotificationService -->|зовнішній API| EmailProvider
    AuthService -.->|опційно: IIdentityProvider| IdentityProvider

    %% --- БД ---
    DataAccess -->|IDbConnection| Database
```

## Примітки

* `Order Service` реалізує бізнес-логіку Завдання 4: резерв `ORDER_ITEM.quantity`, перехід `Pending → Paid/Unpaid`, запис `OrderPaidEvent` в `outbox` в одній транзакції, повтори (1 + 3) і DLQ.
* `Outbox Relay Worker` зчитує `outbox` після коміту транзакції й ініціює `Notification Service`; збій відправки не відкочує оплату.
* `Order Expiry Scheduler` реалізує EARS-12: `Pending` понад 24 год → `Cancelled` + повернення залишків.
* `Identity Provider` позначено пунктиром як опційна інтеграція (у поточному обсязі автентифікація — за `USER.password_hash`, EARS-11).