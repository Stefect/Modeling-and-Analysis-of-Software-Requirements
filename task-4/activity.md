# Activity-діаграма: оформлення та оплата замовлення

Доріжки: `Customer`, `WebApp`, `OrderService` (включно з фоновим Outbox Relay), `PaymentGateway`.
Поза межами цієї діаграми: повторна оплата з `Unpaid` (UC_RetryPayment, EARS-13), повернення коштів, повтори запитів до шлюзу при тайм-ауті.

```mermaid
flowchart TB
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
        O4[Створити ORDER Pending, ORDER_ITEM, резерв]
        O5(( fork ))
        O6[Database.updateOrder status = Paid]
        O7[Database.insertOutbox OrderPaidEvent]
        O8(( join: коміт транзакції ))
        O9[Database.updateOrder status = Unpaid]
        O10[Database.releaseStock]
        O11[[Відповідь: статус замовлення]]
        R1[Relay: зчитати запис outbox]
        R2[Надіслати email покупцю]
        R3{Доставлено?}
        R4[Повтор: +1, +5, +15 хв, разом 4 спроби]
        R5[[status = FAILED, журнал, DLQ]]
    end

    subgraph PAY["PaymentGateway"]
        P1[charge orderId, total_amount]
        P2{approved?}
    end

    C1 --> W1
    W1 -- ні --> W2 --> C3
    W1 -- так --> O1 --> O2
    O2 -- ні --> O3 --> C3
    O2 -- так --> O4 --> C2
    C2 -- ні --> O11 --> C3
    C2 -- так --> P1 --> P2
    P2 -- approved --> O5
    O5 --> O6 --> O8
    O5 --> O7 --> O8
    O8 --> R1 --> R2 --> R3
    R3 -- так --> C3
    R3 -- ні --> R4 --> R2
    R4 -- вичерпано --> R5
    P2 -- declined --> O9 --> O10 --> O11
```
