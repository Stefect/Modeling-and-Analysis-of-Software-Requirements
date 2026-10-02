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

---

## 3. User Stories & Gherkin Scenarios

### US-01: Оформлення замовлення з фіксацією ціни (Price Snapshot)

**Як** `Customer`,  
**я хочу** оформлювати замовлення з фіксацією ціни на момент купівлі,  
**щоб** вартість товарів у моєму чеку не змінювалася у разі оновлення прайсу продавцем.

| Крок | Ключове слово | Сценарій / Дія | Сутності & Атрибути |
| :--- | :--- | :--- | :--- |
| **Precondition** | **Given** | Покупець `Customer` авторизований у системі | `USER` (`user_id`) |
| **Context** | **And** | У кошику є товар `PRODUCT` із ціною 500 UAH | `PRODUCT` (`price` = 500) |
| **Action** | **When** | Покупець натискає кнопку "Оформити замовлення" | `UC_CreateOrder` |
| **Outcome 1** | **Then** | Система створює новий запис `ORDER` зі статусом `Pending` | `ORDER` (`status` = 'Pending') |
| **Outcome 2** | **And** | Фіксує знімок ціни у сутності `ORDER_ITEM` | `ORDER_ITEM` (`unit_price` = 500) |
| **Outcome 3** | **And** | Зменшує доступну кількість товарних залишків | `PRODUCT` (`stock_quantity`) |

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