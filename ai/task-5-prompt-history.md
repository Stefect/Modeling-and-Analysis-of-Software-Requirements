# Журнал промптів — Завдання 5: Архітектура (Component Diagram)

## Промпт 1 (студент)
> Проаналізуй артефакти попередніх завдань (task-1 ERD, task-2 EARS/Use Cases, task-3 sequence, task-4 Activity/Outbox) і запропонуй компонентну модель мультивендорного маркетплейсу. Вкажи компоненти, їхню відповідальність, сутності ERD і Use Cases, які вони покривають, та зовнішні інтеграції. Не пропонуй мікросервіси — залишайся в межах модульного моноліту.

**Відповідь AI:** запропоновано 13 компонентів (Web Client, API Gateway, Auth, Catalog, Seller, Order, Review, Payment Adapter, Notification, Outbox Relay, Expiry Scheduler, Data Access, Database) і 3 зовнішні системи.

**Фідбек студента:** прийнято; уточнено, що `Outbox Relay` і `Expiry Scheduler` мають бути явними компонентами, а не частиною `Order Service`.

## Промпт 2 (студент)
> Згенеруй UML Component Diagram у PlantUML для запропонованої моделі. Для кожного зв'язку вкажи provided/required-інтерфейс (IOrderService, IOrderRepository, IPaymentGateway тощо). Додай пакети (Client, API, Services, Infra, Data, External). Відобрази Outbox і автоскасування (EARS-12).

**Відповідь AI:** згенеровано `component.puml` з інтерфейсами-льодяниками.

**Фідбек студента:** виправлено напрям залежності `OutboxRelay ..> IEmailSender` (relay викликає Notification Service, а не навпаки); додано `IIdentityProvider` як опційний.

## Промпт 3 (студент)
> Перевір діаграму на узгодженість із task-2 (EARS-11: автентифікація за password_hash) і task-4 (Outbox, повтори, DLQ). Познач пунктиром усі опційні інтеграції, яких немає в EARS. Підготуй Mermaid-версію для рендеру на GitHub.

**Відповідь AI:** створено `component.md` з `flowchart LR` і легендою; `Identity Provider` позначено пунктиром.

**Фідбек студента:** прийнято; додано таблицю відповідальності та розділ Out of Scope у `spec.md`.