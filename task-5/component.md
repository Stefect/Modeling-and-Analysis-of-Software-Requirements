# Component Diagram: архітектура маркетплейсу (Завдання 5)

Діаграма подана як **оглядова** компонентна модель системи (спрощено для читабельності). Детальні provided/required-інтерфейси для кожного зв'язку наведено в таблиці `spec.md` §4. Еталонна повна UML-версія з усіма інтерфейсами (льодяники/розетки) — у `task-5/component.puml`.

## Компоненти та залежності (огляд)

Використано горизонтальне розташування шарів (зліва направо), щоб полотно було ширшим і зручнішим для читання. Внутрішні блоки розташовано вертикально (колонками). Стрілки — ортогональні (строго під прямим кутом). Зовнішні системи та інфраструктура винесено в окремі блоки, щоб не накладатися на основні сервіси. Довгі підписи на стрілках перенесено на кілька рядків.

```mermaid
%%{init: {'flowchart': {'curve': 'step'}}}%%
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
        direction TB
        AuthService["Auth Service"]
        CatalogService["Catalog Service"]
        SellerService["Seller Service"]
        OrderService["Order Service"]
        ReviewService["Review Service"]
        NotificationService["Notification Service"]
    end

    %% --- Інфраструктура ---
    subgraph Infra["Інфраструктура"]
        direction TB
        PaymentAdapter["Payment Adapter"]
        OutboxRelay["Outbox Relay Worker"]
        ExpiryScheduler["Order Expiry Scheduler"]
        DataAccess["Data Access (Repository)"]
    end

    %% --- Дані ---
    subgraph Data["Дані"]
        Database[("PostgreSQL")]
    end

    %% --- Зовнішні ---
    subgraph External["Зовнішні системи"]
        direction TB
        PaymentGateway["Payment Gateway"]
        EmailProvider["Email / SMS Provider"]
        IdentityProvider["Identity Provider (опц.)"]
    end

    %% --- Зв'язки (основний потік) ---
    WebClient -->|HTTP/REST| Gateway
    Gateway -->|IAuthService| AuthService
    Gateway -->|ICatalogService| CatalogService
    Gateway -->|IOrderService| OrderService
    Gateway -->|ISellerService| SellerService
    Gateway -->|IReviewService| ReviewService

    AuthService -->|IUserRepository| DataAccess
    CatalogService -->|IProductRepository,<br>ICategoryRepository,<br>IStoreRepository| DataAccess
    SellerService -->|IStoreRepository,<br>IProductRepository| DataAccess
    OrderService -->|IOrderRepository,<br>IProductRepository,<br>IOutboxRepository| DataAccess
    ReviewService -->|IReviewRepository| DataAccess

    OrderService -->|IOutboxRepository| OutboxRelay
    OutboxRelay -->|IOutboxRepository| DataAccess
    OutboxRelay -->|IEmailSender| NotificationService
    ExpiryScheduler -->|IOrderService| OrderService

    OrderService -->|IPaymentGateway| PaymentAdapter
    PaymentAdapter -->|зовнішній API| PaymentGateway
    NotificationService -->|зовнішній API| EmailProvider
    AuthService -.->|IIdentityProvider опц.| IdentityProvider

    DataAccess -->|IDbConnection| Database
```

## Примітки

* `Order Service` реалізує бізнес-логіку Завдання 4: резерв, перехід `Pending → Paid/Unpaid`, запис `OrderPaidEvent` в `outbox`, повтори (1 + 3) і DLQ.
* `Outbox Relay` зчитує `outbox` після коміту і ініціює `Notification Service`; збій відправки не відкочує оплату.
* `Expiry Scheduler` реалізує EARS-12: `Pending` понад 24 год → `Cancelled` + повернення залишків.
* `Identity Provider` позначено пунктиром як опційна інтеграція (автентифікація за `USER.password_hash`, EARS-11).
* Детальні інтерфейси (`IOrderService`, `IOrderRepository`, `IPaymentGateway` тощо) — у `spec.md` §4.
* Еталонна повна версія з усіма інтерфейсами — у `task-5/component.puml`.
