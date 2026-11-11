# Component Diagram: архітектура маркетплейсу (Завдання 5)

Діаграма подана як **оглядова** компонентна модель системи (спрощено для читабельності). Детальні provided/required-інтерфейси для кожного зв'язку наведено в таблиці `spec.md` §4. Еталонна повна UML-версія з усіма інтерфейсами (льодяники/розетки) — у `task-5/component.puml`.

## Компоненти та залежності (огляд)

Використано рендерер `elk` з ортогональними стрілками (`step`) та налаштованими відступами, щоб діаграма була широкою, читабельною і без накладань. База даних винесена в окремий підграф `Дані`. Стрілки підписано повними іменами інтерфейсів з перенесенням рядків.

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk", "curve": "step", "nodeSpacing": 40, "rankSpacing": 60}}}%%
flowchart TB
    %% --- Клієнт ---
    subgraph Client["Клієнт"]
        WebClient["Web Client (SPA)"]
    end

    %% --- API ---
    subgraph API["API"]
        Gateway["API Gateway / BFF"]
    end

    %% --- Прикладні сервіси ---
    subgraph Services["Сервіси"]
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
    subgraph Data["Дані"]
        Database[("PostgreSQL")]
    end

    %% --- Зовнішні системи ---
    subgraph External["Зовнішні системи"]
        IdentityProvider["Identity Provider (опц.)"]
        PaymentGateway["Payment Gateway"]
        EmailProvider["Email / SMS Provider"]
    end

    %% --- Зв'язки Клієнт -> API ---
    WebClient -->|HTTP/REST| Gateway

    %% --- Зв'язки API -> Сервіси ---
    Gateway -->|IAuthService| AuthService
    Gateway -->|ICatalogService| CatalogService
    Gateway -->|IOrderService| OrderService
    Gateway -->|ISellerService| SellerService
    Gateway -->|IReviewService| ReviewService

    %% --- Зв'язки Сервісів ---
    AuthService -->|IUserRepository| DataAccess
    AuthService -.->|IIdentityProvider| IdentityProvider

    CatalogService -->|IProductRepository<br>ICategoryRepository<br>IStoreRepository| DataAccess
    SellerService -->|IStoreRepository<br>IProductRepository| DataAccess

    OrderService -->|IOrderRepository<br>IProductRepository<br>IOutboxRepository| DataAccess
    OrderService -->|IPaymentGateway| PaymentAdapter
    OrderService -->|IOutboxRepository| OutboxRelay

    ReviewService -->|IReviewRepository| DataAccess

    %% --- Зв'язки Інфраструктури ---
    OutboxRelay -->|IOutboxRepository| DataAccess
    OutboxRelay -->|IEmailSender| NotificationService

    ExpiryScheduler -->|IOrderService| OrderService

    PaymentAdapter -->|зовнішній API| PaymentGateway
    NotificationService -->|зовнішній API| EmailProvider

    %% --- БД ---
    DataAccess -->|IDbConnection| Database
```

## Примітки

* `Order Service` реалізує бізнес-логіку Завдання 4: резерв, перехід `Pending → Paid/Unpaid`, запис `OrderPaidEvent` в `outbox`, повтори (1 + 3) і DLQ.
* `Outbox Relay` зчитує `outbox` після коміту і ініціює `Notification Service`; збій відправки не відкочує оплату.
* `Expiry Scheduler` реалізує EARS-12: `Pending` понад 24 год → `Cancelled` + повернення залишків.
* `Identity Provider` позначено пунктиром як опційна інтеграція (автентифікація за `USER.password_hash`, EARS-11).
* Детальні інтерфейси (`IOrderService`, `IOrderRepository`, `IPaymentGateway` тощо) — у `spec.md` §4.
* Еталонна повна версія з усіма інтерфейсами — у `task-5/component.puml`.
