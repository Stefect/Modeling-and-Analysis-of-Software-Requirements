# DEFENSE — Завдання 2: Вимоги та Use Cases

## 1. Намір і критерії

Описати функціональні вимоги маркетплейсу (EARS, user stories, Gherkin, Use Case діаграма), узгоджені з ERD Завдання 1.
Критерії: усі вимоги в EARS; діаграма з генералізацією акторів та `<<include>>`/`<<extend>>`; кожен прецедент прив'язаний до сутностей ERD і має вимогу; матриця трасування N:M з Out of Scope.

## 2. Топ-3 розбіжності (аудит EARS, знайдені в AI-результаті та виправлені)

1. **EARS-03: «блокувати залишки» без механізму.** Розбіжність між EARS і US-01: EARS не описував, як резервується товар. Виправлено: резерв = зменшення `PRODUCT.stock_quantity` на `ORDER_ITEM.quantity`.
   - Коміт: `9111c33` (аудит EARS, оновлено `task-2/spec.md`, критерій 6)
   - Промпт: `ai/task-2-prompt-history.md`, ітерація 11, пункт 1. Spec: `task-2/spec.md`, розд. 4, критерій 6.

2. **EARS-05: «скасовувати резерв» без механізму повернення.** Виправлено: повернення `ORDER_ITEM.quantity` до `PRODUCT.stock_quantity`.
   - Коміт: `9111c33`
   - Промпт: `ai/task-2-prompt-history.md`, ітерація 11, пункт 2. Spec: `task-2/spec.md`, розд. 4, критерій 6.

3. **EARS-06: шкала оцінки не визначена.** Виправлено: `rating` — ціле число від 1 до 5.
   - Коміт: `9111c33`
   - Промпт: `ai/task-2-prompt-history.md`, ітерація 11, пункт 3. Spec: `task-2/spec.md`, розд. 4, критерій 6.

**Додаткові виправлення (ітерація 12, аудит структури):**
- `UC_Auth` не мав вимоги і не був у матриці. Додано EARS-11, US-06, рядок трасування.
- `UC_ManageStore` не мав сценарію для EARS-08. Додано сценарій у US-05.
- Шлях до ERD був хибний (`marketplace-er/spec.md`). Виправлено на `../spec.md`, розд. 2.
- Не було правила для неоплачених `Pending`-замовлень. Додано EARS-12 і `docs/adr/0007`.
- Не було неавторизованого актора. Додано `Anonymous` і `docs/adr/0008`.
- Mermaid не відрізняє генералізацію від зв'язку. Додано еталонну PlantUML-діаграму `task-2/use-case.puml`.
- Посилання на коміти в попередній версії DEFENSE (`2a9dc326`, `0855572` як джерело виправлень) не відповідали змісту. Коміт `2a9dc32` змінював лише `spec.md`, а не DEFENSE. Розбіжності з попередніх ітерацій (оплата як include, генералізація, атрибути ERD) залишаються в журналі AI як історія, а не як топ-3 цього аудиту.

## 3. Ключове рішення і межі сценаріїв

Обрано EARS + Use Case діаграму + Gherkin (`docs/adr/0004`).
- Альтернатива «лише user stories»: двозначність формулювань.
- Альтернатива «лише EARS без діаграми»: не видно залежностей між прецедентами.

**Чому такі межі сценаріїв:**
- `UC_Snapshot` — `<<include>>`, бо фіксація ціни є обов'язковою частиною `UC_CreateOrder` і не має інших варіантів виконання (`docs/adr/0005`).
- `UC_Pay` — `<<extend>>`, бо оплата необов'язкова: замовлення існує і без неї. Оплата як include блокувала б готівку та післяплату (`docs/adr/0005`).
- `UC_SearchCat` — `<<extend>>` до `UC_Search`, бо фільтр є опціональним уточненням пошуку.
- `UC_Auth` — окремий прецедент, а не include в кожному сценарії, бо автентифікація потрібна не всім акторам (пошук доступний анонімно).
- `UC_ExpireOrder` — окремий прецедент з тригером часу, без людського актора; вимагає окремої вимоги EARS-12 (`docs/adr/0007`).
- `User` — абстрактний актор, `Anonymous`, `Customer`, `Seller` — нащадки. Одна автентифікація, різні права (`docs/adr/0006`, `docs/adr/0008`).

## 4. Перевірка: узгодженість із моделлю даних (ERD) і попередньою моделлю

Атрибути й ключі перевірено за `../spec.md` (ERD Завдання 1, розд. 2).

| Сутність | Атрибути (`../spec.md`) | Покриття в Завданні 2 |
| :--- | :--- | :--- |
| USER | id, email, password_hash, first_name, last_name, phone, role, created_at | UC_Auth, EARS-02, EARS-06, EARS-11 |
| STORE | id, owner_id, name, description, created_at | UC_ManageStore, EARS-08, EARS-10 |
| PRODUCT | id, store_id, category_id, title, description, price, stock_quantity, status, created_at | UC_Search, UC_SearchCat, UC_CreateOrder, UC_ManageProduct, UC_ExpireOrder, EARS-01..03, EARS-05, EARS-09, EARS-10, EARS-12 |
| CATEGORY | id, parent_id, name, slug, description | UC_SearchCat, EARS-01 |
| ORDER | id, customer_id, status, total_amount, shipping_address, payment_status, created_at | UC_CreateOrder, UC_Pay, UC_ExpireOrder, EARS-02..05, EARS-12 |
| ORDER_ITEM | id, order_id, product_id, quantity, unit_price | UC_Snapshot (include), EARS-02, EARS-03, EARS-12 |
| REVIEW | id, user_id, product_id, rating, comment, created_at | UC_Review, EARS-06 |

- Обчислювані поля (`STORE.rating`, сума рядка) не зберігаються, згідно з 3NF.
- Нове значення статусу `ORDER.status = Cancelled` (EARS-12) не змінює схему: поле `status` типу `String`.
- EARS-07 (митна вартість) — Out of Scope (`trace-matrix.md`, розд. 4).
- Перевірка покриття: кожен UC має щонайменше одну вимогу (`trace-matrix.md`, розд. 2).

**Відомі ризики (не закриті):**
- Строк 24 години (EARS-12) — припущення; його треба погодити (`docs/adr/0007`).

**Закриті ризики (ітерація 13):**
- Повторна оплата `Unpaid`: закрито EARS-13. Оплата знову резервує товар; при нестачі — відмова. Сценарії в US-02.
- Ролі `USER.role`: закрито. Допустимі значення `customer` і `seller` (`task-2/spec.md`, критерій 1).
- Mermaid-діаграма показує генералізацію як підписану стрілку; точна нотація — у `task-2/use-case.puml`.
