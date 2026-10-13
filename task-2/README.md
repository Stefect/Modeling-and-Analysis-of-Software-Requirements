# Специфікація вимог та Use Cases маркетплейсу (Завдання 2)

## 1. Актори системи (Actors)

| Актор | Тип | Опис |
| :--- | :--- | :--- |
| **User** | Abstract | Абстрактний базовий актор для автентифікованих користувачів. |
| **Customer** | Primary | Покупець, який шукає товари, оформлює замовлення та залишає відгуки. |
| **Seller** | Primary | Продавець, який керує магазином STORE та каталогом PRODUCT. |
| **Payment Gateway** | External | Зовнішня платіжна система для обробки онлайн-транзакцій. |

---

## 2. Функціональні вимоги у форматі EARS

- **EARS-01 (Ubiquitous):** Система SHALL надавати можливість пошуку та фільтрації товарів `PRODUCT` за ієрархічними категоріями `CATEGORY`.
- **EARS-02 (Event-driven):** WHEN покупець `Customer` підтверджує замовлення, система SHALL створювати запис `ORDER` та фіксувати знімок ціни `unit_price` у сутності `ORDER_ITEM`.
- **EARS-03 (State-driven):** WHILE замовлення перебуває у статусі `Pending`, система SHALL блокувати відповідну кількість товарних залишків `stock_quantity`.
- **EARS-04 (Optional):** WHERE покупець обирає онлайн-оплату, система SHALL ініціювати транзакцію через `Payment Gateway`.
- **EARS-05 (Unwanted Behavior):** IF `Payment Gateway` повертає статус відмови транзакції, THEN система SHALL скасовувати резерв товарів та залишати `ORDER` у статусі `Unpaid`.
- **EARS-06 (Event-driven):** WHEN покупець `Customer` залишає відгук `REVIEW`, система SHALL зберігати оцінку та коментар з прив'язкою до `PRODUCT`.
- **EARS-08 (Event-driven):** WHEN продавець `Seller` створює або редагує `STORE`, система SHALL зберігати `name` і `description` з прив'язкою `owner_id` до `USER`.
- **EARS-09 (Event-driven):** WHEN продавець `Seller` додає або змінює `PRODUCT`, система SHALL зберігати `title`, `price`, `stock_quantity` з прив'язкою `store_id` до `STORE`, що належить продавцю.
- **EARS-10 (Unwanted Behavior):** IF продавець намагається змінити `STORE` або `PRODUCT`, який йому не належить (`STORE.owner_id` ≠ `USER.id`), THEN система SHALL відхиляти зміну.
- **EARS-07 (Optional, Out of Scope):** WHERE товар доставляється за межі внутрішнього ринку, система SHALL розраховувати міжнародну митну вартість і збори. *Статус: Out of Scope, див. `trace-matrix.md`, розділ 3.*

---

## 3. User Stories & Gherkin Scenarios

### US-01: Оформлення замовлення з фіксацією ціни (Price Snapshot)

**Як** `Customer`,  
**я хочу** оформлювати замовлення з фіксацією ціни на момент купівлі,  
**щоб** вартість товарів у моєму чеку не змінювалася у разі оновлення прайсу продавцем.

```gherkin
Feature: Оформлення замовлення з фіксацією ціни (EARS-02, EARS-03)

  Scenario: Створення замовлення з Price Snapshot і резервуванням залишку
    Given покупець Customer авторизований у системі (USER.id заповнений)
    And у кошику є товар PRODUCT з price = 500 UAH і stock_quantity >= 1
    When покупець натискає "Оформити замовлення" (UC_CreateOrder)
    Then створюється ORDER зі status = "Pending" і customer_id = USER.id
    And створюється ORDER_ITEM з unit_price = 500 UAH
    And PRODUCT.stock_quantity зменшується на ORDER_ITEM.quantity

  Scenario: Зміна ціни товару після оформлення не впливає на замовлення
    Given існує ORDER_ITEM з unit_price = 500 UAH
    When продавець змінює PRODUCT.price на 600 UAH
    Then ORDER_ITEM.unit_price залишається 500 UAH
```

**Acceptance:** після оформлення ціна в чеку не змінюється, навіть якщо продавець оновив прайс.

---

### US-02: Онлайн-оплата замовлення (extend) та скасування резерву при відмові

**Як** `Customer`,  
**я хочу** оплатити замовлення онлайн або обрати інший спосіб оплати,  
**щоб** оплата не блокувала оформлення замовлення і не залишала резерв при відмові банку.

```gherkin
Feature: Онлайн-оплата як extend до оформлення (EARS-04, EARS-05)

  Scenario: Успішна онлайн-оплата
    Given існує ORDER зі status = "Pending"
    When покупець обирає онлайн-оплату (UC_Pay)
    And Payment Gateway повертає статус "approved"
    Then ORDER.status змінюється на "Paid"
    And ORDER.payment_status = "approved"

  Scenario: Відмова платежу скасовує резерв
    Given існує ORDER зі status = "Pending" і зарезервованими PRODUCT.stock_quantity
    When покупець обирає онлайн-оплату (UC_Pay)
    And Payment Gateway повертає статус "declined"
    Then ORDER.status змінюється на "Unpaid"
    And резерв товарів скасовується (PRODUCT.stock_quantity повертається)

  Scenario: Оплата не обрана (extend не виконується)
    Given існує ORDER зі status = "Pending"
    When покупець не обирає онлайн-оплату
    Then ORDER залишається у status = "Pending"
    And Payment Gateway не викликається
```

**Acceptance:** оформлення замовлення не залежить від оплати; відмова платежу не залишає заблокованих залишків.

---

### US-03: Залишення відгуку на товар

**Як** `Customer`,  
**я хочу** залишити оцінку та коментар до товару,  
**щоб** інші покупці бачили досвід використання товару.

```gherkin
Feature: Відгук на товар (EARS-06)

  Scenario: Збереження відгуку
    Given покупець Customer авторизований у системі
    And товар PRODUCT з id = P1 існує
    When покупець залишає відгук з rating = 5 і comment = "Добре"
    Then створюється REVIEW з user_id = USER.id, product_id = P1, rating = 5
    And REVIEW.comment = "Добре"
```

**Acceptance:** відгук зберігається з прив'язкою до `USER` і `PRODUCT`.

---

### US-04: Пошук товарів з фільтром за ієрархією категорій (EARS-01)

**Як** `Customer`,  
**я хочу** шукати товари у категорії разом із її підкатегоріями,  
**щоб** швидко знаходити потрібний товар.

```gherkin
Feature: Пошук і фільтрація за категоріями (EARS-01)

  Scenario: Фільтр за батьківською категорією включає підкатегорії
    Given існує CATEGORY "Електроніка" з дочірньою CATEGORY "Смартфони" (parent_id = Електроніка.id)
    And існує PRODUCT з category_id = Смартфони.id
    When покупець фільтрує товари за категорією "Електроніка" (UC_SearchCat)
    Then результат містить PRODUCT з category_id = Смартфони.id

  Scenario: Пошук без фільтра
    Given існують товари PRODUCT у різних категоріях
    When покупець виконує пошук за назвою (UC_Search)
    Then результат містить товари, що відповідають запиту
```

---

### US-05: Керування магазином і товарами продавця (EARS-08..10)

**Як** `Seller`,  
**я хочу** додавати та змінювати товари у своєму магазині,  
**щоб** покупці бачили актуальний каталог.

```gherkin
Feature: Керування товарами продавця (EARS-08, EARS-09, EARS-10)

  Scenario: Додавання товару до власного магазину
    Given продавець Seller авторизований і є owner STORE з id = S1
    When продавець додає PRODUCT з title = "Чохол", price = 200 UAH, stock_quantity = 10, store_id = S1
    Then створюється PRODUCT зі store_id = S1
    And PRODUCT.stock_quantity = 10

  Scenario: Спроба змінити товар чужого магазину відхиляється
    Given STORE з id = S2 належить іншому продавцю
    When продавець намагається змінити PRODUCT з store_id = S2
    Then система відхиляє зміну
    And PRODUCT не змінюється
```

**Acceptance:** продавець змінює лише власні магазини та товари.

---

## 4. Use Case Model

### 4.1. Специфікація прецедентів використання (Таблиця)

| Use Case ID | Назва прецеденту | Актор | Тип зв'язку | Опис / Пов'язаний UC |
| :--- | :--- | :--- | :--- | :--- |
| **UC_Auth** | Авторизуватися в системі | `User` | Primary | Базова автентифікація користувача. |
| **UC_Search** | Шукати товари | `Customer` | Primary | Пошук товарів у каталозі `PRODUCT`. |
| **UC_SearchCat** | Фільтрувати за `CATEGORY` | `Customer` | Generalization | Розширення пошуку через фільтри категорій. |
| **UC_CreateOrder**| Оформити замовлення `ORDER` | `Customer` | Primary | Створення замовлення зі статусом `Pending`. |
| **UC_Snapshot** | Зафіксувати ціну `unit_price`| System | `<<include>>` | **Обов'язковий:** фіксація ціни в `ORDER_ITEM`. |
| **UC_Pay** | Оплатити через шлюз | `Payment Gateway`| `<<extend>>` | **Опціональний:** онлайн-оплата після створення. |
| **UC_Review** | Залишити відгук `REVIEW` | `Customer` | Primary | Оцінка придбаного товару. |
| **UC_ManageStore**| Керувати магазином `STORE` | `Seller` | Primary | Налаштування профілю магазину продавця. |
| **UC_ManageProduct**| Управляти товарами `PRODUCT`| `Seller` | Primary | Додавання та редагування товарів. |

---

### 4.2. Use Case Diagram (Компактна вертикальна)

```mermaid
flowchart TD
    %% --- Актори ---
    User([User])
    Customer([Customer])
    Seller([Seller])
    PaymentGateway([Payment Gateway])

    Customer -->|успадковує| User
    Seller -->|успадковує| User

    %% --- Межі системи ---
    subgraph Marketplace ["Мультивендорний Маркетплейс"]
        direction TB
        
        UC_Auth("UC_Auth: Авторизація")
        UC_Search("UC_Search: Пошук товарів")
        UC_CreateOrder("UC_CreateOrder: Оформлення ORDER")
        UC_Snapshot("UC_Snapshot: Фіксація unit_price")
        UC_Pay("UC_Pay: Оплата")
        UC_Review("UC_Review: Відгук REVIEW")
        UC_ManageStore("UC_ManageStore: Магазин STORE")
        UC_ManageProduct("UC_ManageProduct: Товари PRODUCT")

        %% Примусовий вертикальний порядок
        UC_Auth ~~~ UC_Search
        UC_Search ~~~ UC_CreateOrder
        UC_CreateOrder ~~~ UC_Snapshot
        UC_Snapshot ~~~ UC_Pay
        UC_Pay ~~~ UC_Review
        UC_Review ~~~ UC_ManageStore
        UC_ManageStore ~~~ UC_ManageProduct

        %% Логічні залежності
        UC_CreateOrder -. include .-> UC_Snapshot
        UC_Pay -. extend .-> UC_CreateOrder
    end

    %% --- Зв'язки Акторів ---
    User --> UC_Auth
    Customer --> UC_Search
    Customer --> UC_CreateOrder
    Customer --> UC_Review

    Seller --> UC_ManageStore
    Seller --> UC_ManageProduct

    UC_Pay --> PaymentGateway