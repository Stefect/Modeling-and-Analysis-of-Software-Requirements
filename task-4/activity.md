# Activity-діаграма: оформлення та оплата замовлення

Доріжки: `Customer`, `WebApp`, `OrderService` (включно з фоновим Outbox Relay), `PaymentGateway`.
Стани `ORDER` позначено в мітках вузлів: `Pending` (створення), `Paid` (успішна оплата), `Unpaid` (відмова оплати). Стан `Cancelled` поза цією діаграмою (EARS-12, лише з `Pending`).
Поза межами цієї діаграми: повторна оплата з `Unpaid` (UC_RetryPayment, EARS-13), повернення коштів, повтори запитів до шлюзу при тайм-ауті.
```mermaid
flowchart TB
    START((старт))

    subgraph CUST["Customer"]
        C1([Оформити замовлення])
        C2{Онлайн-оплата?}
        C3([Переглянути статус])
    end

    subgraph WEB["WebApp"]
        W1{Сесія дійсна?<br/>EARS-14}
        W2[[401 Unauthorized]]
    end

    subgraph ORD["OrderService"]
        O1[Прочитати ціни<br/>та залишки]
        O2{Товар є<br/>на складі?}
        O3[[insufficientStock]]
        O4[Створити ORDER<br/>status = Pending]
        O10[Передати на оплату]
        subgraph TX["Транзакція оплати"]
            O5((fork))
            O6[ORDER.status<br/>= Paid]
            O7[insert outbox<br/>OrderPaidEvent]
            O8((join: коміт))
        end
        O9[status = Unpaid<br/>резерв знято]
        O11[[Статус замовлення]]
        R1[Relay: читати outbox]
        R2[Надіслати email]
        R3{Доставлено?}
        R4[Спроба +1]
        R5[[FAILED, DLQ]]
    end

    subgraph PAY["PaymentGateway"]
        P1[Списати кошти]
        P2{Схвалено?}
    end

    STOP((кінець))
    UNP((Unpaid))
    PEND((Pending))

    START --> C1 --> W1
    W1 -- ні --> W2 --> C3
    W1 -- так --> O1 --> O2
    O2 -- ні --> O3 --> C3
    O2 -- так --> O4 --> C2
    C2 -- ні --> PEND
    C2 -- так --> O10 --> P1 --> P2
    P2 -- approved --> O5
    O5 --> O6 --> O8
    O5 --> O7 --> O8
    O8 --> R1 --> R2 --> R3
    R3 -- так --> C3
    R3 -- ні --> R4
    R4 -- спроба < 4 --> R2
    R4 -- спроба ≥ 4 --> R5 --> C3
    P2 -- declined --> O9 --> O11
    O11 --> C3
    O9 --> UNP
    C3 --> STOP
```