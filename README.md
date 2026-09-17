# Концептуальна модель даних маркетплейсу

## Опис системи
Система для організації мультивендорного маркетплейсу, що забезпечує взаємодію покупців, магазинів продавців, каталог товарів з ієрархічними категоріями, оформлення замовлень та систему відгуків.

## ER-діаграма (v2 - Нормалізована модель 3NF)

```mermaid
erDiagram
    CATEGORY ||--o{ CATEGORY : "parent of"
    CATEGORY ||--o{ PRODUCT : classifies
    STORE ||--o{ PRODUCT : offers
    USER ||--o{ STORE : owns
    USER ||--o{ ORDER : places
    USER ||--o{ REVIEW : writes
    ORDER ||--|{ ORDER_ITEM : contains
    PRODUCT ||--o{ ORDER_ITEM : "included in"
    PRODUCT ||--o{ REVIEW : receives

    CATEGORY {
        uuid id PK
        uuid parent_id FK
        string name
        string slug
        string description
    }

    STORE {
        uuid id PK
        uuid owner_id FK
        string name
        string description
        datetime created_at
    }

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