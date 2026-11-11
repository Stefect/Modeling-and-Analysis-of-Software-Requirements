# Component Diagram: архітектура маркетплейсу (Завдання 5)

Діаграма подана як **оглядова** компонентна модель системи (спрощено для читабельності). Детальні provided/required-інтерфейси для кожного зв'язку наведено в таблиці `spec.md` §4. Еталонна повна UML-версія з усіма інтерфейсами (льодяники/розетки) — у `task-5/component.puml`.

## Компоненти та залежності (огляд)

Використано вертикальне розташування шарів (зверху вниз), щоб стрілки були ортогональними та не накладалися. Внутрішні блоки розташовано горизонтально (рядками), щоб діаграма була ширшою. Стрілки — строго під прямим кутом (без заокруглень). Довгі назви інтерфейсів на стрілках скорочено.

```mermaid
%%{init: {"flowchart": {"curve": "step"}}}%%
flowchart TB
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
        direction LR
        AuthService["Auth Service"]
        CatalogService["Catalog Service"]
        SellerService["Seller Service"]
        OrderService["Order Service"]
        ReviewService["Review Service"]
    end

    %% --- Воркери та адаптери (теж інфраструктура, але верхній шар) ---
    subgraph Workers["Фонові процеси та Адаптери"]
        direction LR
        NotificationService["Notification Service"]
        ExpiryScheduler["Order Expiry Scheduler"]
        PaymentAdapter["Payment Adapter"]
        OutboxRelay["Outbox Relay Worker"]
    end

    %% --- Зовнішні системи ---
    subgraph External["Зовнішні системи"]
        direction LR
        IdentityProvider["Identity Provider (опц.)"]
        PaymentGateway["Payment Gateway"]
        EmailProvider["Email / SMS Provider"]
    end

    %% --- База та доступ до неї ---
    subgraph DAL["Доступ до даних (Інфраструктура)"]
        DataAccess["Data Access (Repository)"]
    end

    subgraph Data["Дані"]
        Database[("PostgreSQL")]
    end

    %% ==========================================
    %% ЗВ'ЯЗКИ (Згори донизу)
    %% ==========================================

    WebClient -->|HTTP/REST| Gateway

    Gateway -->|IAuth| AuthService
    Gateway -->|ICatalog| CatalogService
    Gateway -->|ISeller| SellerService
    Gateway -->|IOrder| OrderService
    Gateway -->|IReview| ReviewService

    AuthService -.->|IIdentityProvider| IdentityProvider
    PaymentAdapter -->|API| PaymentGateway
    NotificationService -->|API| EmailProvider

    ExpiryScheduler -->|IOrderService| OrderService
    OrderService -->|IPaymentGateway| PaymentAdapter
    OrderService -->|IOutbox| OutboxRelay
    OutboxRelay -->|IEmailSender| NotificationService

    %% Зв'язки до репозиторію (йдуть строго вниз)
    AuthService -->|IUser| DataAccess
    CatalogService -->|IProduct, ICategory, IStore| DataAccess
    SellerService -->|IStore, IProduct| DataAccess
    OrderService -->|IOrder, IProduct, IOutbox| DataAccess
    ReviewService -->|IReview| DataAccess
    OutboxRelay -->|IOutbox| DataAccess

    DataAccess -->|IDbConnection| Database
```

## Примітки

* `Order Service` реалізує бізнес-логіку Завдання 4: резерв, перехід `Pending → Paid/Unpaid`, запис `OrderPaidEvent` в `outbox`, повтори (1 + 3) і DLQ.
* `Outbox Relay` зчитує `outbox` після коміту і ініціює `Notification Service`; збій відправки не відкочує оплату.
* `Expiry Scheduler` реалізує EARS-12: `Pending` понад 24 год → `Cancelled` + повернення залишків.
* `Identity Provider` позначено пунктиром як опційна інтеграція (автентифікація за `USER.password_hash`, EARS-11).
* Детальні інтерфейси (`IOrderService`, `IOrderRepository`, `IPaymentGateway` тощо) — у `spec.md` §4.
* Еталонна повна версія з усіма інтерфейсами — у `task-5/component.puml`.
