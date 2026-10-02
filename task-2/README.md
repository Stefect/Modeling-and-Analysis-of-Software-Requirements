```markdown
# Вимоги та Use Cases маркетплейсу (Завдання 2)

## 1. EARS-вимоги
- **EARS-01 (Ubiquitous):** Система SHALL надавати можливість пошуку товарів за назвою та категорією.
- **EARS-02 (Event-driven):** WHEN покупець натискає "Оформити замовлення", система SHALL створювати запис замовлення `ORDER`.
- **EARS-03 (State-driven):** WHILE замовлення перебуває в статусі "Очікує оплати", система SHALL блокувати зміну складу товарів.
- **EARS-04 (Optional):** WHERE додано промокод, система SHALL застосовувати знижку до суми замовлення.

## 2. User Stories & Gherkin
- **US-01:** Як `Customer`, я хочу оформлювати замовлення з кошика, щоб придбати обрані товари.
  - **Gherkin:**
    - **Given** покупець додав товари в кошик
    - **When** покупець підтверджує замовлення
    - **Then** система створює замовлення в статусі `Pending`

## 3. Use Case Diagram
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