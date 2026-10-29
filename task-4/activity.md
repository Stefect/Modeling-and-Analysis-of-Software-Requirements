# Activity-діаграма: оформлення та оплата замовлення

Діаграма подана однією лінійною схемою, що читається зверху вниз у порядку виконання процесу. Роль виконавця вказано в тексті вузла (префікс `Customer:`, `WebApp:`, `OrderService:`, `PaymentGateway:`). Групування за доріжками не використано.

Стани `ORDER`: `Pending` (створення), `Paid` (успішна оплата), `Unpaid` (відмова оплати). Стан `Cancelled` поза цією діаграмою (EARS-12, лише з `Pending`).
Поза межами цієї діаграми: повторна оплата з `Unpaid` (UC_RetryPayment, EARS-13), повернення коштів, повтори запитів до шлюзу при тайм-ауті.

```mermaid
flowchart TB
    START((старт)) --> C1([Customer: Оформити замовлення])
    C1 --> W1{WebApp: сесія дійсна?<br/>EARS-14}
    W1 -- ні --> W2[[WebApp: 401 Unauthorized]]
    W2 --> C3([Customer: Переглянути статус])
    W1 -- так --> O1[OrderService: прочитати ціни<br/>та залишки]
    O1 --> O2{OrderService: товар є<br/>на складі?}
    O2 -- ні --> O3[[insufficientStock]]
    O3 --> C3
    O2 -- так --> O4[OrderService: створити ORDER<br/>status = Pending]
    O4 --> C2{Customer: онлайн-оплата?}
    C2 -- ні --> PEND((Pending: автоскасування, UC_ExpireOrder))
    C2 -- так --> O10[OrderService: передати на оплату]
    O10 --> P1[PaymentGateway: списати кошти]
    P1 --> P2{PaymentGateway: схвалено?}
    P2 -- approved --> F1((fork))
    F1 --> O6[OrderService: ORDER.status = Paid]
    F1 --> O7[OrderService: insert outbox<br/>OrderPaidEvent]
    O6 --> J1((join: коміт))
    O7 --> J1
    J1 --> R1[OrderService: Relay читає outbox]
    R1 --> R2[OrderService: надіслати email]
    R2 --> R3{Доставлено?}
    R3 -- так --> C3
    R3 -- ні --> R4[Спроба +1]
    R4 -- спроба < 4 --> R2
    R4 -- спроба ≥ 4 --> R5[[FAILED, DLQ]]
    R5 --> C3
    P2 -- declined --> O9[OrderService: status = Unpaid<br/>резерв знято]
    O9 --> O11[[Статус замовлення]]
    O11 --> C3
    O9 --> UNP((Unpaid: повтор оплати, UC_RetryPayment))
    C3 --> STOP((кінець))
```
