# Журнал взаємодії з AI (Prompt History) — Завдання 2

## Ітерація 1: Первинна генерація вимог та Use Case діаграми

**Промт:**
> "На основі сутностей з Завдання 1 (USER, STORE, PRODUCT, ORDER, ORDER_ITEM, REVIEW) згенеруй для мультивендорного маркетплейсу:
> 1. Набір вимог EARS для покупця (Customer), продавця (Seller) та платіжного шлюзу (Payment Gateway).
> 2. User Stories та Gherkin-сценарії.
> 3. Use Case діаграму в Mermaid з акторами та зв'язками include, extend та generalization."

**Отриманий результат від AI:**

### EARS-вимоги:
- **EARS-01:** Користувач може шукати товари за назвою.
- **EARS-02:** WHEN покупець тисне "Купити", система SHALL створити замовлення.
- **EARS-03:** WHILE система обробляє платіж, Payment Gateway SHALL підтвердити його.

### Use Case Diagram (Mermaid):
```mermaid
flowchart TD
    Customer([Customer])
    Seller([Seller])
    PaymentGateway([Payment Gateway])

    subgraph Marketplace ["Маркетплейс"]
        UC1(Шукати товари)
        UC2(Оформити замовлення)
        UC3(Оплатити замовлення)
        UC4(Керувати каталогом)
    end

    Customer --> UC1
    Customer --> UC2
    Customer --> UC3
    Seller --> UC4
    UC3 --> PaymentGateway
    UC2 .->|include| UC3