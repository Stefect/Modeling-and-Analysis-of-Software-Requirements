# Специфікація архітектури: Component Diagram маркетплейсу (Завдання 5)

## 1. Намір і критерії

Ця специфікація описує архітектуру мультивендорного маркетплейсу у вигляді UML Component Diagram. Вона узгоджується з ER-моделлю (Завдання 1), вимогами та Use Cases (Завдання 2), sequence-діаграмами (Завдання 3) й Activity-діаграмою процесу оформлення й оплати замовлення (Завдання 4). Артефакт має показати компоненти, їхні provided/required-інтерфейси, залежності між компонентами та зовнішні інтеграції.

**Критерії прийняття (фіксуються комітом):**

1. **Покриття домену:** кожен компонент відповідає за підмножину сутностей ERD (`USER`, `STORE`, `CATEGORY`, `PRODUCT`, `ORDER`, `ORDER_ITEM`, `REVIEW`) і/або Use Cases (`UC_Auth`, `UC_Search`, `UC_CreateOrder`, `UC_Pay`, `UC_ExpireOrder`, `UC_Review`, `UC_ManageStore`, `UC_ManageProduct`).
2. **Інтерфейси:** для кожного зв'язку між компонентами вказано provided/required-інтерфейс (наприклад `IOrderService`, `IOrderRepository`), а не «виклик класу».
3. **Зовнішні інтеграції:** `Payment Gateway`, постачальник email/SMS та (опційно) `Identity Provider` ізольовані адаптерами; прямі виклики з доменних сервісів до зовнішніх API заборонені.
4. **Узгодженість із процесом:** компоненти підтримують потік Завдання 4 — резерв/оплата, Outbox Relay, повтори (1 + 3), DLQ, автоскасування через 24 год (EARS-12).
5. **Транзакційність:** зміни `PRODUCT.stock_quantity` і `ORDER.status` виконуються в межах однієї БД-транзакції через `Data Access`; паралельні замовлення не призводять до від'ємних залишків (NFR-07).
6. **Форма артефакту:** декларативний опис у PlantUML (`component.puml`) + рендер у Mermaid (`component.md`); обидва синтаксично валідні.

## 2. Компоненти та відповідальність

| Компонент | Відповідальність | Сутності / Use Cases |
|---|---|---|
| `Web Client (SPA)` | Інтерфейс покупця й продавця | UC_Search, UC_CreateOrder, UC_Pay, UC_Review, UC_Manage* |
| `API Gateway / BFF` | Єдина точка входу, маршрутизація, перевірка сесії (EARS-14) | Усі захищені UC |
| `Auth Service` | Автентифікація, сесії, ролі | `USER`, UC_Auth |
| `Catalog Service` | Пошук, категорії, залишки | `PRODUCT`, `CATEGORY`, `STORE`, UC_Search, UC_SearchCat |
| `Seller Service` | Керування магазином і товарами | `STORE`, `PRODUCT`, UC_ManageStore, UC_ManageProduct |
| `Order Service` | Оформлення, резерв, оплата, скасування | `ORDER`, `ORDER_ITEM`, UC_CreateOrder, UC_Pay, UC_ExpireOrder |
| `Review Service` | Відгуки | `REVIEW`, UC_Review |
| `Payment Adapter` | Ізоляція зовнішнього платіжного шлюзу | UC_Pay (EARS-04, EARS-05) |
| `Notification Service` | Відправка email/SMS | NFR-16, EARS-16 |
| `Outbox Relay Worker` | Асинхронне надсилання подій з outbox | Завдання 4 (EARS-15, EARS-16) |
| `Order Expiry Scheduler` | Автоскасування `Pending` через 24 год | UC_ExpireOrder (EARS-12) |
| `Data Access (Repository)` | Доступ до БД, транзакції | Усі сутності |
| `Database (PostgreSQL)` | Збереження ER-моделі | Усі сутності |

## 3. Зовнішні інтеграції

* `Payment Gateway` (зовнішня) — через `Payment Adapter` (`IPaymentGateway`).
* `Email/SMS Provider` (зовнішня) — через `Notification Service` (`IEmailSender`, `ISmsSender`).
* `Identity Provider (OAuth/OIDC)` (зовнішня, опційно) — через `Auth Service` (`IIdentityProvider`); у поточному обсязі автентифікація виконується за `USER.password_hash` (EARS-11).

## 4. Інтерфейси (provided / required)

* `Web Client` requires: `IAuthApi`, `ICatalogApi`, `IOrderApi`, `IReviewApi`, `ISellerApi`.
* `API Gateway` provides: `IAuthApi`, `ICatalogApi`, `IOrderApi`, `IReviewApi`, `ISellerApi`; requires: `IAuthService`, `ICatalogService`, `IOrderService`, `IReviewService`, `ISellerService`.
* `Order Service` requires: `IOrderRepository`, `IProductRepository`, `IPaymentGateway`, `IOutboxRepository`.
* `Data Access` provides: `IOrderRepository`, `IProductRepository`, `IUserRepository`, `IStoreRepository`, `IReviewRepository`, `IOutboxRepository`; requires: `IDbConnection`.
* `Outbox Relay Worker` requires: `IOutboxRepository`, `IEmailSender`.

## 5. Поза обсягом (Out of Scope)

* Розгортання в Kubernetes / багатовузловий кластер — поза межами курсу.
* Окремий пошуковий рушій (Elasticsearch) — не моделюється; пошук виконується в БД.
* Кешування (Redis) — може бути додано пізніше без зміни інтерфейсів.
* Адміністративна роль / backoffice — не моделюється (див. Завдання 2).

## 6. Критерії перевірки

1. Кожен компонент зі специфікації присутній на діаграмі.
2. Кожен зовнішній виклик ізольовано адаптером.
3. Стан `ORDER` (`Pending`, `Paid`, `Unpaid`, `Cancelled`) і потік Outbox відображені компонентами `Order Service`, `Outbox Relay Worker`.
4. Діаграма рендериться без синтаксичних помилок (Mermaid на GitHub, PlantUML локально).