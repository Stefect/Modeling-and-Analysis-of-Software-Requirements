# Activity-діаграма: оформлення та оплата замовлення

Діаграма подана лінійно, слайдами в порядку виконання процесу. Кожен слайд є окремою схемою з одним кроком процесу. Роль виконавця вказано в тексті вузла (префікс `Customer:`, `WebApp:`, `OrderService:`, `PaymentGateway:`). Перехід до наступного слайда позначено вузлом `Далі: слайд N`.

Стани `ORDER`: `Pending` (створення), `Paid` (успішна оплата), `Unpaid` (відмова оплати). Стан `Cancelled` поза цією діаграмою (EARS-12, лише з `Pending`).
Поза межами цієї діаграми: повторна оплата з `Unpaid` (UC_RetryPayment, EARS-13), повернення коштів, повтори запитів до шлюзу при тайм-ауті.

### Слайд 1. Оформлення та перевірка сесії

```mermaid
flowchart TB
    START((старт)) --> C1([Customer: Оформити замовлення])
    C1 --> W1{WebApp: сесія дійсна?<br/>EARS-14}
    W1 -- ні --> W2[[WebApp: 401 Unauthorized]]
    W2 --> C3([Customer: Переглянути статус])
    C3 --> STOP((кінець))
    W1 -- так --> NEXT1[[Далі: слайд 2]]
```

### Слайд 2. Перевірка товару та створення замовлення

```mermaid
flowchart TB
    IN2[[Далі з слайда 1: сесія дійсна]] --> O1[OrderService: прочитати ціни та залишки]
    O1 --> O2{OrderService: товар є на складі?}
    O2 -- ні --> O3[[insufficientStock]]
    O3 --> C3b([Customer: Переглянути статус])
    O2 -- так --> O4[OrderService: створити ORDER<br/>status = Pending]
    O4 --> C2{Customer: онлайн-оплата?}
    C2 -- ні --> PEND((Pending: автоскасування, UC_ExpireOrder))
    C2 -- так --> NEXT2[[Далі: слайд 3]]
```

### Слайд 3. Списання коштів

```mermaid
flowchart TB
    IN3[[Далі з слайда 2: онлайн-оплата = так]] --> O10[OrderService: передати на оплату]
    O10 --> P1[PaymentGateway: списати кошти]
    P1 --> P2{PaymentGateway: схвалено?}
    P2 -- approved --> NEXT3[[Далі: слайд 4]]
    P2 -- declined --> NEXT6[[Далі: слайд 6]]
```

### Слайд 4. Фіксація оплати (атомарна транзакція)

```mermaid
flowchart TB
    IN4[[Далі з слайда 3: approved]] --> F1((fork))
    F1 --> O6[OrderService: ORDER.status = Paid]
    F1 --> O7[OrderService: insert outbox<br/>OrderPaidEvent]
    O6 --> J1((join: коміт))
    O7 --> J1
    J1 --> NEXT5[[Далі: слайд 5]]
```

### Слайд 5. Фонове сповіщення (Outbox Relay)

```mermaid
flowchart TB
    IN5[[Далі з слайда 4: коміт]] --> R1[OrderService: Relay читає outbox]
    R1 --> R2[OrderService: надіслати email]
    R2 --> R3{Доставлено?}
    R3 -- так --> C3c([Customer: Переглянути статус])
    R3 -- ні --> R4[Спроба +1]
    R4 -- спроба < 4 --> R2
    R4 -- спроба ≥ 4 --> R5[[FAILED, DLQ]]
    R5 --> C3d([Customer: Переглянути статус])
```

### Слайд 6. Відмова оплати

```mermaid
flowchart TB
    IN6[[Далі з слайда 3: declined]] --> O9[OrderService: status = Unpaid<br/>резерв знято]
    O9 --> O11[[Статус замовлення]]
    O11 --> C3e([Customer: Переглянути статус])
    O9 --> UNP((Unpaid: повтор оплати, UC_RetryPayment))
```
