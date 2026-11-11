# Component Diagram: архітектура маркетплейсу (Завдання 5)

Діаграма подана як **оглядова** компонентна модель системи (спрощено для читабельності). Детальні provided/required-інтерфейси для кожного зв'язку наведено в таблиці `spec.md` §4. Еталонна повна UML-версія з усіма інтерфейсами (льодяники/розетки) — у `task-5/component.puml`.

## Компоненти та залежності (огляд)

Використано рендерер `elk` з ортогональними стрілками (`step`) та налаштованими відступами, щоб діаграма була широкою, читабельною і без накладань. Внутрішні блоки розташовано горизонтально (рядками). Довгі назви інтерфейсів на стрілках скорочено.

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk", "curve": "step", "nodeSpacing": 40, "rankSpacing": 60}}}%%
flowchart TB
    %% --- Клієнт та API ---
    WebClient["Web Client (SPA)"] -->|HTTP/REST| Gateway["API Gateway / BFF"]

    %% --- Центральна бізнес-логіка (Сервіси) ---
    subgraph Services["Сервіси"]
        AuthService["Auth Service"]
        CatalogService["Catalog Service"]
        SellerService["Seller Service"]
        OrderService["Order Service"]
        ReviewService["Review Service"]
    end

    Gateway -->|IAuth| AuthService
    Gateway -->|ICatalog| CatalogService
    Gateway -->|ISeller| SellerService
    Gateway -->|IOrder| OrderService
    Gateway -->|IReview| ReviewService

    %% --- Фонові процеси та Адаптери ---
    subgraph Workers["Фонові процеси та Адаптери"]
        ExpiryScheduler["Order Expiry Scheduler"]
        PaymentAdapter["Payment Adapter"]
        OutboxRelay["Outbox Relay Worker"]
        NotificationService["Notification Service"]
    end

    ExpiryScheduler -->|IOrderService| OrderService
    OrderService -->|IPaymentGateway| PaymentAdapter
    OrderService -->|IOutbox| OutboxRelay
    OutboxRelay -->|IEmailSender| NotificationService

    %% --- Рівень Даних (DAL) ---
    subgraph DAL["Доступ до даних"]
        DataAccess["Data Access (Repository)"]
    end

    AuthService -->|IUser| DataAccess
    CatalogService -->|IProduct, ICategory, IStore| DataAccess
    SellerService -->|IStore, IProduct| DataAccess
    OrderService -->|IOrder, IProduct, IOutbox| DataAccess
    ReviewService -->|IReview| DataAccess
    OutboxRelay -->|IOutbox| DataAccess

    %% --- База даних ---
    subgraph Data["Дані"]
        Database[("PostgreSQL")]
    end
    DataAccess -->|IDbConnection| Database

    %% --- Зовнішні системи ---
    subgraph External["Зовнішні системи"]
        IdentityProvider["Identity Provider (опц.)"]
        PaymentGateway["Payment Gateway"]
        EmailProvider["Email / SMS Provider"]
    end

    AuthService -.->|IIdentityProvider| IdentityProvider
    PaymentAdapter -->|API| PaymentGateway
    NotificationService -->|API| EmailProvider
```

## Примітки

* `Order Service` реалізує бізнес-логіку Завдання 4: резерв, перехід `Pending → Paid/Unpaid`, запис `OrderPaidEvent` в `outbox`, повтори (1 + 3) і DLQ.
* `Outbox Relay` зчитує `outbox` після коміту і ініціює `Notification Service`; збій відправки не відкочує оплату.
* `Expiry Scheduler` реалізує EARS-12: `Pending` понад 24 год → `Cancelled` + повернення залишків.
* `Identity Provider` позначено пунктиром як опційна інтеграція (автентифікація за `USER.password_hash`, EARS-11).
* Детальні інтерфейси (`IOrderService`, `IOrderRepository`, `IPaymentGateway` тощо) — у `spec.md` §4.
* Еталонна повна версія з усіма інтерфейсами — у `task-5/component.puml`.
