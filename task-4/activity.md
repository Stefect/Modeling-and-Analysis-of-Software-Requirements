# Activity-діаграма: оформлення та оплата замовлення

Доріжки: `Customer`, `WebApp`, `OrderService` (включно з фоновим Outbox Relay), `PaymentGateway`.
Стани `ORDER` позначено в мітках вузлів: `Pending` (створення), `Paid` (успішна оплата), `Unpaid` (відмова оплати). Стан `Cancelled` поза цією діаграмою (EARS-12, лише з `Pending`).
Поза межами цієї діаграми: повторна оплата з `Unpaid` (UC_RetryPayment, EARS-13), повернення коштів, повтори запитів до шлюзу при тайм-ауті.
```mermaid
flowchart TB
    START((старт))

    subgraph CUST["Customer"]
        C1([Натиснути «Оформити замовлення»])
        C2{Обрано онлайн-оплату?}
        C3([Переглянути статус замовлення])
    end

    subgraph WEB["WebApp"]
        W1{Сесія дійсна? EARS-14}
        W2[[Відповідь 401]]
    end

    subgraph ORD["OrderService"]
        O1[Прочитати ціни та залишки UC_Snapshot]
        O2{stock_quantity ≥ ORDER_ITEM.quantity?}
        O3[[Відмова insufficientStock]]
        O4[Створити ORDER status = Pending, ORDER_ITEM, резерв]
        O10[Перенаправити на PaymentGateway]
        subgraph TX["Транзакція оплати (атомарна)"]
            O5(( fork ))
            O6[Database.updateOrder status = Paid]
            O7[Database.insertOutbox OrderPaidEvent]
            O8(( join: коміт ))
        end
        O9[Database.updateOrder status = Unpaid і releaseStock в одній транзакції]
        O11[[Відповідь: статус замовлення]]
        R1[Relay: зчитати запис outbox після коміту]
        R2[Надіслати email покупцю]
        R3{Доставлено?}
        R4[Зафіксувати спробу +1]
        R5[[status = FAILED, журнал, DLQ]]
    end

    subgraph PAY["PaymentGateway"]
        P1[charge orderId, total_amount]
        P2{approved?}
    end

    STOP((кінець))
    UNP((Unpaid: кінець процесу, повтор оплати — UC_RetryPayment))
    PEND((Pending: кінець процесу, автоскасування — UC_ExpireOrder))

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
    R4 -- вичерпано (спроба ≥ 4) --> R5 --> C3
    P2 -- declined --> O9 --> O11
    O9 --> UNP
    C3 --> STOP
```