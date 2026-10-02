# Специфікація вимог та Use Cases маркетплейсу (Завдання 2)

## 1. Актори системи (Actors)

| Актор | Тип | Опис |
| :--- | :--- | :--- |
| **User** | Abstract | Абстрактний базовий актор для автентифікованих користувачів. |
| **Customer** | Primary | Покупець, який шукає товари, оформлює замовлення та залишає відгуки. |
| **Seller** | Primary | Продавець, який керує магазином `STORE` та каталогом `PRODUCT`. |
| **Payment Gateway** | External | Зовнішня платіжна система для обробки онлайн-транзакцій. |

---

## 2. Функціональні вимоги у форматі EARS

- **EARS-01 (Ubiquitous):** Система SHALL надавати можливість пошуку та фільтрації товарів `PRODUCT` за ієрархічними категоріями `CATEGORY`.
- **EARS-02 (Event-driven):** WHEN покупець `Customer` підтверджує замовлення, система SHALL створювати запис `ORDER` та фіксувати знімок ціни `unit_price` у сутності `ORDER_ITEM`.
- **EARS-03 (State-driven):** WHILE замовлення перебуває у статусі `Pending`, система SHALL блокувати відповідну кількість товарних залишків `stock_quantity`.
- **EARS-04 (Optional):** WHERE покупець обирає онлайн-оплату, система SHALL ініціювати транзакцію через `Payment Gateway`.
- **EARS-05 (Unwanted Behavior):** IF `Payment Gateway` повертає статус відмови транзакції, THEN система SHALL скасовувати резерв товарів та залишати `ORDER` у статусі `Unpaid`.
- **EARS-06 (Event-driven):** WHEN покупець `Customer` залишає відгук `REVIEW`, система SHALL зберігати оцінку та коментар з прив'язкою до `PRODUCT`.

---

## 3. User Stories & Gherkin Scenarios

### US-01: Оформлення замовлення з фіксацією ціни (Price Snapshot)
**Як** `Customer`,  
**я хочу** оформлювати замовлення з фіксацією ціни на момент купівлі,  
**щоб** вартість товарів у моєму чеку не змінювалася у разі оновлення прайсу продавцем.

**Сценарій Gherkin:**
- **Given** покупець `Customer` авторизований у системі
- **And** у кошику є товар `PRODUCT` із ціною 500 UAH
- **When** покупець натискає кнопку "Оформити замовлення"
- **Then** система створює новий запис `ORDER` зі статусом `Pending`
- **And** створює позицію `ORDER_ITEM` із зафіксованим `unit_price` = 500 UAH
- **And** зменшує доступний `stock_quantity` товару на вказану кількість

---

## 4. Use Case Diagram (v2 - Уточнена)

```mermaid
flowchart TD
    User([User])
    Customer([Customer])
    Seller([Seller])
    PaymentGateway([Payment Gateway])

    Customer --|> User
    Seller --|> User

    subgraph Marketplace ["Мультивендорний Маркетплейс"]
        UC_Auth(Авторизуватися в системі)
        UC_Search(Шукати товари)
        UC_SearchCat(Фільтрувати за CATEGORY)
        UC_CreateOrder(Оформити замовлення ORDER)
        UC_Snapshot(Зафіксувати ціну unit_price)
        UC_Pay(Оплатити через шлюз)
        UC_Review(Залишити відгук REVIEW)
        UC_ManageStore(Керувати магазином STORE)
        UC_ManageProduct(Управляти товарами PRODUCT)
    end

    User --> UC_Auth
    Customer --> UC_Search
    Customer --> UC_CreateOrder
    Customer --> UC_Review
    
    Seller --> UC_ManageStore
    Seller --> UC_ManageProduct

    UC_SearchCat --|> UC_Search

    UC_CreateOrder .->|"<<include>>"| UC_Snapshot
    UC_Pay .->|"<<extend>>"| UC_CreateOrder

    UC_Pay --> PaymentGateway