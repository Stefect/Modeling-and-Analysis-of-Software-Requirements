# Специфікація взаємодії: оформлення замовлення (Завдання 3)

## 1. Намір та критерії
Ця специфікація деталізує динаміку виконання Use Case `UC_CreateOrder` (актор `Customer`) та пов'язаних з ним процесів із Завдання 2. Взаємодія моделюється за допомогою 8 окремих sequence-діаграм (S1–S8), кожна з яких покриває конкретний сценарій Gherkin з `task-2/README.md`. Модель узгоджена з EARS-вимогами Завдання 2 та сутностями за текстом Завдання 1 (`USER`, `PRODUCT`, `ORDER`, `ORDER_ITEM`).

Критерії (фіксуються комітом):
1. **Покриття сценаріїв:** Створено 8 окремих sequence-діаграм (S1–S8) для сценаріїв US-01 та US-02. Автоматичне скасування через 24 години (`UC_ExpireOrder`) свідомо винесено за межі цієї роботи, оскільки воно ініціюється фоновим таймером системи, а не користувачем.
2. **Учасники:** Актор `Customer`, клієнтський застосунок `WebApp`, сервіси (`UserService`, `CatalogService`, `OrderService`), сховище `Database` (таблиці `USER`, `PRODUCT`, `ORDER`, `ORDER_ITEM`) та зовнішня система `PaymentGateway`.
3. **Строгий формалізм викликів та EARS-мітки:** Усі повідомлення є сигнатурами методів із параметрами. Опис дій прозою виключено. Кожен виклик між сервісами містить посилання на відповідну вимогу EARS (`EARS-02..05`, `EARS-13`, `EARS-14`); внутрішні виклики до `Database` реалізують ці вимоги на рівні збереження даних.
4. **Комбіновані фрагменти:** У кожній діаграмі присутній щонайменше один фрагмент: `alt` (гілки авторизації/оплати), `opt` (опціональна онлайн-оплата), `loop` (ітерація по позиціях кошика), `break` (достроковий вихід при недостачі товару) або `critical` (атомарні транзакції БД).
5. **Схема деактивації та обмеження Mermaid:** Виклики створюють активацію (`->>+`). Через особливості лінійної деактивації Mermaid, повернення результатів (наприклад, `OrderService -->> WebApp` у S1) та повернення помилок у гілках `alt`/`break` (401 у S1, `insufficientStock` у S1 та S6) виконуються пунктирними стрілками, а деактивація сервісів явно закривається через `deactivate` після відповідного фрагмента.
6. **Відповідність EARS:** Створено еталонні версії у Mermaid (`sequence.md`) та PlantUML (`sequence.puml`). При успішній оплаті статуси змінюються викликом `Database.updateOrder(orderId, status = 'Paid', payment_status = 'approved')` (S5). При відмові оплати надсилаються два окремих виклики: `Database.releaseStock(productId, qty)` у циклі `loop` та `Database.updateOrder(orderId, status = 'Unpaid', payment_status = 'declined')` (S3, S5).

## 2. Учасники взаємодії
| Учасник | Тип | Роль та відповідність моделі |
| :--- | :--- | :--- |
| `Customer` | Актор | Користувач, нащадок `User` (Завдання 2) |
| `WebApp` | Компонент UI | Клієнтський веб-інтерфейс (поза ERD) |
| `UserService` | Сервіс | Аутентифікація, перевірка сесій та ролей (читає `USER`) |
| `CatalogService` | Сервіс | Перевірка ціни та залишків (читає `PRODUCT`) |
| `OrderService` | Сервіс | Бізнес-логіка створення замовлень, знімків цін та резерву |
| `Database` | Сховище | Реляційна БД (таблиці `USER`, `PRODUCT`, `ORDER`, `ORDER_ITEM`) |
| `PaymentGateway` | Зовнішня система | Еквайринг для онлайн-оплати (`Payment Gateway`) |

## 3. Основний потік та виконання сценаріїв
1. **Ініціалізація та перевірка сесії (EARS-14):** `Customer` надсилає `WebApp.confirmOrder(sessionToken, cartItems)`. `WebApp` звертається до `UserService.validateSession(sessionToken)`. `UserService` викликає `Database.getSession(sessionToken)` та `Database.getUserRole(userId)`. При недійсній сесії повертається відмова 401.
2. **Отримання актуальних даних (UC_Snapshot, EARS-02):** `OrderService` через `CatalogService.getProductList(productIds)` запитує з `Database.findProducts(productIds)` актуальні ціни та `stock_quantity`.
3. **Перевірка залишків (EARS-03):** У гілці `alt`: якщо `stock_quantity < ORDER_ITEM.quantity`, повертається відмова `insufficientStock(productId)` і створення замовлення переривається.
4. **Резервування та транзакційність (EARS-03):** У блоці `critical` `OrderService` виконує виклики `Database.insertOrder(userId, totalAmount, 'Pending')` та `Database.insertOrderItems(orderId, items, unit_price)`. Далі в циклі `loop` для кожної позиції викликається `Database.reserveStock(productId, qty)`.
5. **Онлайн-оплата (UC_Pay, EARS-04, EARS-05):** У блоці `opt` виконується виклик `PaymentGateway.charge(orderId, total_amount)`. При успіху `OrderService` викликає `Database.updateOrder(orderId, status = 'Paid', payment_status = 'approved')`. При відмові у транзакційному блоці `critical` викликається `Database.releaseStock(productId, qty)` для кожної позиції, після чого `OrderService` викликає `Database.updateOrder(orderId, status = 'Unpaid', payment_status = 'declined')`.

## 4. Обмеження та межі (Out of Scope)
- Автоматичне скасування замовлень через 24 години (EARS-12) виконується окремим фоновим демоном `System Timer` і не входить до даного Use Case.
- Сценарій S7 демонструє вплив виклику `CatalogService.updatePrice(...)` продавцем на вже створене замовлення: фіксований `ORDER_ITEM.unit_price` не змінюється при зміні ціни товару в каталозі.