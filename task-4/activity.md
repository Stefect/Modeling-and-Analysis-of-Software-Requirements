# Activity-діаграма: оформлення та оплата замовлення

Діаграма подана п'ятьма інлайн-блоками в порядку виконання процесу. Перехід між блоками позначено вузлами `→ Доріжка: дія` (вихід) і `← Доріжка: дія` (вхід). Блоки 3 і 5 належать одній доріжці `OrderService`, але розділені етапами: створення замовлення та фіксація оплати.

Доріжки: `Customer`, `WebApp`, `OrderService` (включно з фоновим Outbox Relay), `PaymentGateway`.
Стани `ORDER` позначено в мітках вузлів: `Pending` (створення), `Paid` (успішна оплата), `Unpaid` (відмова оплати). Стан `Cancelled` поза цією діаграмою (EARS-12, лише з `Pending`).
Поза межами цієї діаграми: повторна оплата з `Unpaid` (UC_RetryPayment, EARS-13), повернення коштів, повтори запитів до шлюзу при тайм-ауті.

### Крок 1. Customer: оформлення

```mermaid
flowchart TB
    START((старт)) --> C1([Оформити замовлення])
    C1 --> C1out[[→ WebApp: перевірка сесії]]

    C2in[[← OrderService: ORDER створено, Pending]] --> C2{Онлайн-оплата?}
    C2 -- ні --> PEND((Pending))
    C2 -- так --> C2out[[→ OrderService: Передати на оплату]]

    W2in[[← WebApp: 401 Unauthorized]] --> C3([Переглянути статус])
    OSin1[[← OrderService: insufficientStock]] --> C3
    OSin2[[← OrderService: статус замовлення]] --> C3
    OSin3[[← OrderService: доставлено]] --> C3
    OSin4[[← OrderService: FAILED, DLQ]] --> C3
    C3 --> STOP((кінець))
```

### Крок 2. WebApp: перевірка сесії

```mermaid
flowchart TB
    W0in[[← Customer: Оформити замовлення]] --> W1{Сесія дійсна?<br/>EARS-14}
    W1 -- ні --> W2[[401 Unauthorized]]
    W2 --> W2out[[→ Customer: Переглянути статус]]
    W1 -- так --> W1out[[→ OrderService: Прочитати ціни та залишки]]
```

### Крок 3. OrderService: створення замовлення

```mermaid
flowchart TB
    O0in[[← WebApp: сесія дійсна]] --> O1[Прочитати ціни<br/>та залишки]
    O1 --> O2{Товар є<br/>на складі?}
    O2 -- ні --> O3[[insufficientStock]]
    O3 --> O3out[[→ Customer: Переглянути статус]]
    O2 -- так --> O4[Створити ORDER<br/>status = Pending]
    O4 --> O4out[[→ Customer: Онлайн-оплата?]]

    O10in[[← Customer: Онлайн-оплата = так]] --> O10[Передати на оплату]
    O10 --> O10out[[→ PaymentGateway: Списати кошти]]
```

### Крок 4. PaymentGateway: списання

```mermaid
flowchart TB
    P_in[[← OrderService: Передати на оплату]] --> P1[Списати кошти]
    P1 --> P2{Схвалено?}
    P2 -- approved --> P2ok[[→ OrderService: approved]]
    P2 -- declined --> P2no[[→ OrderService: declined]]
```

### Крок 5. OrderService: фіксація оплати, сповіщення, відмова

```mermaid
flowchart TB
    P_ok[[← PaymentGateway: approved]] --> O5((fork))
    O5 --> O6[ORDER.status<br/>= Paid]
    O5 --> O7[insert outbox<br/>OrderPaidEvent]
    O6 --> O8((join: коміт))
    O7 --> O8
    O8 --> R1[Relay: читати outbox]
    R1 --> R2[Надіслати email]
    R2 --> R3{Доставлено?}
    R3 -- так --> R3out[[→ Customer: Переглянути статус]]
    R3 -- ні --> R4[Спроба +1]
    R4 -- спроба < 4 --> R2
    R4 -- спроба ≥ 4 --> R5[[FAILED, DLQ]]
    R5 --> R5out[[→ Customer: Переглянути статус]]

    P_no[[← PaymentGateway: declined]] --> O9[status = Unpaid<br/>резерв знято]
    O9 --> O11[[Статус замовлення]]
    O11 --> O11out[[→ Customer: Переглянути статус]]
    O9 --> UNP((Unpaid))
```
