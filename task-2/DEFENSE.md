# DEFENSE — Завдання 2: Вимоги та Use Cases

## 1. Намір і критерії

Описати функціональні вимоги маркетплейсу (EARS, user stories, Gherkin, Use Case діаграма), узгоджені з ERD Завдання 1.
Критерії: усі вимоги в EARS; діаграма з генералізацією акторів та `<<include>>`/`<<extend>>`; прецеденти прив'язані до сутностей ERD; матриця трасування з Out of Scope.

## 2. Топ-3 розбіжності (знайдені в AI-результаті та виправлені)

1. **Оплата як обов'язковий include.** AI зробив оплату частиною створення замовлення. Виправлено: оплата — `<<extend>>`, фіксація ціни — `<<include>>` (ітерація 2).
   - Коміт: [2a9dc326](https://github.com/Stefect/Modeling-and-Analysis-of-Software-Requirements/commit/2a9dc326f89c664d97e10b3d11e990f5a2517f94)
   - Промпт: `ai/task-2-prompt-history.md`, ітерація 2. Spec: `task-2/spec.md`, розд. 3–4.

2. **Відсутність генералізації акторів.** `Customer` і `Seller` були незалежними. Виправлено: абстрактний актор `User` (ітерація 2).
   - Коміт: [2a9dc326](https://github.com/Stefect/Modeling-and-Analysis-of-Software-Requirements/commit/2a9dc326f89c664d97e10b3d11e990f5a2517f94)
   - Промпт: `ai/task-2-prompt-history.md`, ітерація 2. Spec: `task-2/spec.md`, розд. 3–4.

3. **Вимоги без сутностей БД.** EARS описували поведінку без атрибутів ERD. Виправлено: у вимогах явно вказано сутності та атрибути (ітерація 5).
   - Коміт: [0855572](https://github.com/Stefect/Modeling-and-Analysis-of-Software-Requirements/commit/08555728487477056a7dd2b247e903c427577683)
   - Промпт: `ai/task-2-prompt-history.md`, ітерація 5. Spec: `task-2/README.md`, розд. 2.

## 3. Ключове рішення

Обрано EARS + Use Case діаграму + Gherkin (`docs/adr/0004`).
- Альтернатива «лише user stories»: двозначність формулювань.
- Альтернатива «лише EARS без діаграми»: не видно залежностей між прецедентами.

Оплата — `<<extend>>`, ціна — `<<include>>` (`docs/adr/0005`). Оплата як include блокує готівку та післяплату. Ціна фіксується при створенні замовлення, а не при оплаті.

`User` — абстрактний актор (`docs/adr/0006`): одна автентифікація, рольові права для `Customer` і `Seller`.

## 4. Перевірка: узгодженість із моделлю даних (ERD)

Атрибути й ключі перевірено за `marketplace-er/spec.md`.

| Сутність | Атрибути (spec.md) | Покриття в Завданні 2 |
| :--- | :--- | :--- |
| USER | id, email, password_hash, first_name, last_name, phone, role, created_at | UC_Auth, EARS-02, EARS-06 |
| STORE | id, owner_id, name, description, created_at | UC_ManageStore, EARS-08, EARS-10 |
| PRODUCT | id, store_id, category_id, title, description, price, stock_quantity, status, created_at | UC_Search, UC_SearchCat, UC_CreateOrder, UC_ManageProduct, EARS-01..03, EARS-09, EARS-10 |
| CATEGORY | id, parent_id, name, slug, description | UC_SearchCat, EARS-01 |
| ORDER | id, customer_id, status, total_amount, shipping_address, payment_status, created_at | UC_CreateOrder, UC_Pay, EARS-02..05 |
| ORDER_ITEM | id, order_id, product_id, quantity, unit_price | UC_Snapshot (include), EARS-02 |
| REVIEW | id, user_id, product_id, rating, comment, created_at | UC_Review, EARS-06 |

- Обчислювані поля (`STORE.rating`, сума рядка) не зберігаються, згідно з 3NF.
- EARS-07 (митна вартість) — Out of Scope (`trace-matrix.md`, розд. 3).

Прецеденти продавця (UC_ManageStore, UC_ManageProduct) закрито EARS-08..10 і US-05. Вимоги спираються на `owner_id` і `store_id` з ERD.
