# Журнал взаємодії з AI (Prompt History)

## Ітерація 1: Початковий запит та базовий монолітний магазин
**Промт:**
> "Мені потрібно спроєктувати базу даних для онлайн-магазину на Mermaid. Зроби таблиці користувачів, категорій, товарів, замовлень і відгуків."

**Отриманий результат від AI:**
```mermaid
erDiagram
    CATEGORY ||--o{ CATEGORY : "parent of"
    CATEGORY ||--o{ PRODUCT : contains
    USER ||--o{ ORDER : places
    ORDER ||--|{ ORDER_ITEM : includes
    PRODUCT ||--o{ ORDER_ITEM : ordered
    USER ||--o{ REVIEW : writes
    PRODUCT ||--o{ REVIEW : reviewed

    USER {
        uuid id PK
        string email
        string password_hash
        string full_name
        string phone
        string role
        datetime created_at
    }

    CATEGORY {
        uuid id PK
        uuid parent_id FK
        string title
        string slug
    }

    PRODUCT {
        uuid id PK
        uuid category_id FK
        string title
        string sku
        decimal price
        int stock
        datetime created_at
    }

    ORDER {
        uuid id PK
        uuid user_id FK
        string status
        decimal total_amount
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

---

## Ітерація 2: Перехід до мультивендорності (Prom.ua)
**Промт:**
> "Ми змінюємо домен з простого магазину на мультивендорний маркетплейс (як Prom.ua). Потрібно додати підтримку магазинів (STORES), розбити ім'я користувача на first_name/last_name, додати доставку та оновити схему."

**Отриманий результат від AI:**
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

## Ітерація 3: Інженерний аудит студента та виявлення дефектів AI

**Зауваження та критика роботи моделі:**
1. **Порушення 3NF (аномалії надлишковості):** AI додав обчислюване поле `ORDER_ITEMS.subtotal` (`quantity * unit_price`) та динамічний агрегат `STORES.rating`. Це порушує 3-ю нормальну форму.
2. **Невідповідність конвенціям іменування:** Сутності згенеровані у множині (`USERS`, `STORES`, `PRODUCTS` замість `USER`, `STORE`, `PRODUCT`).
3. **Невалідний синтаксис типів:** Використано типи `text`, `timestamp` та непідтримуваний у Mermaid модифікатор `UK`.

**Коригувальний промт:**
> "Проведи нормалізацію до 3NF: видали subtotal та rating. Приведи сутності до однини, типи до string/datetime та прибери UK. Зафіксуй кардинальність замовлення як мінімум 1 позиція."

**Отриманий результат від AI:**
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

---

## Ітерація 4: Оптимізація Mermaid-топології (усунення перетинів ліній)

**Проблема рендерингу:** 
Зв'язок `USER -> REVIEW` перетинав усю схему зверху вниз, викликаючи зламаний автолейаут у GitHub.

**Коригувальний промт / Власноручне виправлення:**
> "Перегрупуй послідовність сутностей та зв'язків Mermaid зверху вниз (CATEGORY -> STORE -> USER -> PRODUCT -> ORDER -> ORDER_ITEM -> REVIEW), щоб лінії зв'язків не перетинали таблиці."

**Фінальна стабільна схема:**
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

---

## Ітерація 5: Формалізація архітектурних рішень (ADR)
**Промт:**
> "Сформуй пакет архітектурних записів рішень (ADR за стандартом MADR):
> 1. ADR 0001: Виділення сутності STORE для мультивендорності.
> 2. ADR 0002: Фіксація unit_price у позиціях замовлення як знімка ціни.
> 3. ADR 0003: Рекурсивний зв'язок категорій (parent_id)."

**Отриманий результат:**
Створено документацію в директорії `docs/adr/` (`0001-separate-store-entity.md`, `0002-price-snapshot-in-order-items.md`, `0003-hierarchical-categories-self-reference.md`).

---

## Ітерація 6: Підготовка документа захисту (DEFENSE.md)
**Промт:**
> "Згенеруй документ DEFENSE.md строго за структурою DoD: намір і критерії, топ-3 виправлення зі специфікацією, ключові архітектурні рішення та чекліст узгодженості."

**Отриманий результат:**
Створено фінальний артефакт `DEFENSE.md` для захисту лабораторної роботи.