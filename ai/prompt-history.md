# Журнал взаємодії з AI (Prompt History)

## Ітерація 1: Базовий драфт
**Промт:**
> "Мені потрібно спроєктувати ER-модель бази даних для маркетплейсу. Зроби базову структуру: користувачі (покупці та продавці), товари, категорії, замовлення, позиції замовлення та відгуки. Опиши сутності, атрибути та зв'язки у синтаксисі Mermaid erDiagram."

**Отриманий результат:**
```mermaid
erDiagram
    USERS ||--o{ STORES : "owns"
    USERS ||--o{ ORDERS : "places"
    USERS ||--o{ REVIEWS : "writes"

    STORES ||--o{ PRODUCTS : "offers"

    CATEGORIES ||--o{ CATEGORIES : "parent of"
    CATEGORIES ||--o{ PRODUCTS : "classifies"

    PRODUCTS ||--o{ ORDER_ITEMS : "included in"
    PRODUCTS ||--o{ REVIEWS : "receives"

    ORDERS ||--|{ ORDER_ITEMS : "contains"

    USERS {
        uuid id PK
        string email UK
        string password_hash
        string first_name
        string last_name
        string phone
        string role
        timestamp created_at
    }

    STORES {
        uuid id PK
        uuid owner_id FK
        string name
        string description
        string rating
        timestamp created_at
    }

    CATEGORIES {
        uuid id PK
        uuid parent_id FK
        string name
        string slug UK
        text description
    }

    PRODUCTS {
        uuid id PK
        uuid store_id FK
        uuid category_id FK
        string title
        text description
        decimal price
        int stock_quantity
        string status
        timestamp created_at
    }

    ORDERS {
        uuid id PK
        uuid customer_id FK
        string status
        decimal total_amount
        string shipping_address
        string payment_status
        timestamp created_at
    }

    ORDER_ITEMS {
        uuid id PK
        uuid order_id FK
        uuid product_id FK
        int quantity
        decimal unit_price
        decimal subtotal
    }

    REVIEWS {
        uuid id PK
        uuid user_id FK
        uuid product_id FK
        int rating
        text comment
        timestamp created_at
    }

```
---

## Ітерація 2: Інженерний аудит та Spec-First виправлення
**Промт / Фідбек:**
> "Я провів аудит моделі:
> 1. Видалити надлишкове обчислюване поле subtotal із ORDER_ITEMS та rating зі STORES (дотримання 3NF).
> 2. Зафіксувати назви сутностей в однині (USER, STORE, PRODUCT, ORDER, CATEGORY, ORDER_ITEM, REVIEW).
> 3. Замінити типи text на string, timestamp на datetime, прибрати нестандартні модифікатори UK.
> 4. Додати чіткі критерії прийняття (Acceptance Criteria) у spec.md та зафіксувати unit_price для історії цін.
> Онови ER-модель згідно з цими правилами."

**Отриманий результат:**
```mermaid
erDiagram
    CATEGORY ||--o{ CATEGORY : "parent of"
    CATEGORY ||--o{ PRODUCT : classifies
    STORE ||--o{ PRODUCT : offers
    USER ||--o{ STORE : owns
    
    USER ||--o{ ORDER : places
    ORDER ||--|{ ORDER_ITEM : contains
    PRODUCT ||--o{ ORDER_ITEM : "included in"

    PRODUCT ||--o{ REVIEW : receives
    USER ||--o{ REVIEW : writes

    USER {
        uuid id PK
        string email
        string password_hash
        string first_name
        string last_name
        string phone
        string role
        datetime created_at
    }

    STORE {
        uuid id PK
        uuid owner_id FK
        string name
        string description
        datetime created_at
    }

    CATEGORY {
        uuid id PK
        uuid parent_id FK
        string name
        string slug
        string description
    }

    PRODUCT {
        uuid id PK
        uuid store_id FK
        uuid category_id FK
        string title
        string description
        decimal price
        int stock_quantity
        string status
        datetime created_at
    }

    ORDER {
        uuid id PK
        uuid customer_id FK
        string status
        decimal total_amount
        string shipping_address
        string payment_status
        datetime created_at
    }

    ORDER_ITEM {
        uuid id PK
        uuid order_id FK
        uuid product_id FK
        int quantity
        decimal unit_price
    }

    REVIEW {
        uuid id PK
        uuid user_id FK
        uuid product_id FK
        int rating
        string comment
        datetime created_at
    }

```