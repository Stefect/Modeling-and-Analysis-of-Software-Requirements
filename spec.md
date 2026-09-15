# Специфікація моделі даних: Маркетплейс

## 1. Критерії прийняття (Acceptance Criteria)
- **Нормалізація (3NF):** Відсутність дублювання даних і транзитивних залежностей; обчислювані поля (сума рядка замовлення, рейтинг) не зберігаються в базі даних.
- **Ідентифікатори:** Усі сутності мають первинний ключ типу `UUID`.
- **Фіксація цін:** Історична вартість товару фіксується в сутності `ORDER_ITEM` атрибутом `unit_price`.
- **Кардинальність замовлень:** Замовлення обов'язково містить мінімум одну позицію (`1..*`).
- **Мультивендорність:** Розділення облікового запису користувача (`USER`) та магазину (`STORE`).

## 2. Сутності та атрибути
- **USER:** `id` (UUID, PK), `email` (String), `password_hash` (String), `first_name` (String), `last_name` (String), `phone` (String), `role` (String), `created_at` (DateTime).
- **STORE:** `id` (UUID, PK), `owner_id` (UUID, FK -> USER), `name` (String), `description` (String), `created_at` (DateTime).
- **CATEGORY:** `id` (UUID, PK), `parent_id` (UUID, FK -> CATEGORY, Optional), `name` (String), `slug` (String), `description` (String).
- **PRODUCT:** `id` (UUID, PK), `store_id` (UUID, FK -> STORE), `category_id` (UUID, FK -> CATEGORY), `title` (String), `description` (String), `price` (Decimal), `stock_quantity` (Int), `status` (String), `created_at` (DateTime).
- **ORDER:** `id` (UUID, PK), `customer_id` (UUID, FK -> USER), `status` (String), `total_amount` (Decimal), `shipping_address` (String), `payment_status` (String), `created_at` (DateTime).
- **ORDER_ITEM:** `id` (UUID, PK), `order_id` (UUID, FK -> ORDER), `product_id` (UUID, FK -> PRODUCT), `quantity` (Int), `unit_price` (Decimal).
- **REVIEW:** `id` (UUID, PK), `user_id` (UUID, FK -> USER), `product_id` (UUID, FK -> PRODUCT), `rating` (Int), `comment` (String), `created_at` (DateTime).

## 3. Зв'язки
- `USER ||--o{ STORE` (1 : 0..N) — користувач володіє магазинами.
- `STORE ||--o{ PRODUCT` (1 : 0..N) — магазин виставляє товари.
- `CATEGORY ||--o{ CATEGORY` (1 : 0..N) — ієрархія категорій.
- `CATEGORY ||--o{ PRODUCT` (1 : 0..N) — категорія класифікує товари.
- `USER ||--o{ ORDER` (1 : 0..N) — користувач розміщує замовлення.
- `ORDER ||--|{ ORDER_ITEM` (1 : 1..N) — замовлення містить від 1 позиції.
- `PRODUCT ||--o{ ORDER_ITEM` (1 : 0..N) — товар входить у позицію замовлення.
- `USER ||--o{ REVIEW` (1 : 0..N) та `PRODUCT ||--o{ REVIEW` (1 : 0..N) — відгуки покупців.