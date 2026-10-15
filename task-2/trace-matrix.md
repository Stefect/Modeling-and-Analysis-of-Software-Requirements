# Матриця трасування вимог (Traceability Matrix) — Завдання 2

## 1. Таблиця покриття (EARS -> Use Cases -> ER-Entities)

| ID вимоги | Тип EARS | Опис вимоги | Пов'язані Use Cases | Сутності ERD (Завдання 1) | Стан покриття |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **EARS-01** | Ubiquitous | Пошук та фільтрація товарів за категоріями | `UC_Search`, `UC_SearchCat` | `PRODUCT`, `CATEGORY` | Повне |
| **EARS-02** | Event-driven | Створення замовлення та фіксація ціни `unit_price` | `UC_CreateOrder`, `UC_Snapshot` | `ORDER`, `ORDER_ITEM` | Повне |
| **EARS-03** | State-driven | Резервування залишків товарів у статусі Pending | `UC_CreateOrder` | `PRODUCT`, `ORDER` | Повне |
| **EARS-04** | Optional | Ініціалізація онлайн-оплати через шлюз | `UC_Pay`, `UC_CreateOrder` | `ORDER` | Повне |
| **EARS-05** | Unwanted Behavior | Скасування резерву при відмові платежу | `UC_Pay`, `UC_CreateOrder` | `ORDER`, `PRODUCT` | Повне |
| **EARS-06** | Event-driven | Збереження відгуку та оцінки товару | `UC_Review` | `REVIEW`, `PRODUCT`, `USER` | Повне |
| **EARS-08** | Event-driven | Створення та редагування магазину з прив'язкою до власника | `UC_ManageStore` | `STORE`, `USER` | Повне |
| **EARS-09** | Event-driven | Додавання та зміна товарів у власному магазині | `UC_ManageProduct` | `PRODUCT`, `STORE` | Повне |
| **EARS-10** | Unwanted Behavior | Відхилення змін магазину або товару чужим продавцем | `UC_ManageStore`, `UC_ManageProduct` | `STORE`, `PRODUCT` | Повне |
| **EARS-11** | Event-driven | Автентифікація анонімного користувача за email і паролем | `UC_Auth` | `USER` | Повне |
| **EARS-12** | Unwanted Behavior | Автоскасування `Pending` без оплати через 24 год із поверненням резерву | `UC_ExpireOrder`, `UC_Pay` | `ORDER`, `PRODUCT` | Повне |
| **EARS-13** | Event-driven | Повторна оплата `Unpaid` із повторним резервом; відмова при нестачі товару | `UC_Pay`, `UC_CreateOrder` | `ORDER`, `PRODUCT`, `ORDER_ITEM` | Повне |
| **EARS-07** | Optional | Розрахунок міжнародної митної вартості та зборів (WHERE доставка за межі ринку) | *Відсутній* | *Відсутні* | **Out of Scope** |
| **NFR-01** | Ubiquitous | Хешування паролів, без відкритого зберігання | `UC_Auth` | `USER` (`password_hash`) | Покрито NFR, перевірка: аудит коду/тест |
| **NFR-02** | Ubiquitous | Передача даних лише через HTTPS (TLS 1.2+) | `UC_Auth`, `UC_CreateOrder`, `UC_Pay`, `UC_ManageStore` | Усі сутності (транспорт) | Покрито NFR, перевірка: конфігурація сервера |
| **NFR-03** | Event-driven | Блокування входу після 5 невдалих спроб за 15 хв | `UC_Auth` | `USER` (інфраструктурний лічильник, ADR-0009) | Покрито NFR, перевірка: тест навантаження входу |
| **NFR-04** | Ubiquitous | Видимість персональних даних лише власнику облікового запису | `UC_Auth`, `UC_Review` | `USER` (`email`, `phone`, `first_name`, `last_name`) | Покрито NFR, перевірка: тест прав доступу |
| **NFR-05** | Ubiquitous | Пошук ≤ 500 мс для 95% запитів при 100 000 товарів | `UC_Search`, `UC_SearchCat` | `PRODUCT`, `CATEGORY` | Покрито NFR, перевірка: навантажувальний тест |
| **NFR-06** | Ubiquitous | Оформлення замовлення ≤ 2 с для 95% запитів (без шлюзу) | `UC_CreateOrder`, `UC_Snapshot` | `ORDER`, `ORDER_ITEM` | Покрито NFR, перевірка: навантажувальний тест |
| **NFR-07** | Ubiquitous | Атомарна зміна `stock_quantity`, без від'ємних значень | `UC_CreateOrder`, `UC_Pay`, `UC_ExpireOrder`, `UC_ManageProduct` | `PRODUCT` | Покрито NFR, перевірка: тест паралельних замовлень |
| **NFR-08** | Ubiquitous | `ORDER.total_amount` = сума `quantity × unit_price` | `UC_CreateOrder`, `UC_Snapshot` | `ORDER`, `ORDER_ITEM` | Покрито NFR, перевірка: тест цілісності |
| **NFR-09** | Ubiquitous | Доступність ≥ 99,5% на місяць | Усі UC (системний рівень) | Усі сутності | Покрито NFR, перевірка: моніторинг |
| **NFR-10** | Event-driven | Журнал аудиту входів, `ORDER.status`, `PRODUCT.stock_quantity` (поза ERD, ADR-0009) | `UC_Auth`, `UC_CreateOrder`, `UC_Pay`, `UC_ExpireOrder`, `UC_ManageProduct` | `USER`, `ORDER`, `PRODUCT` (посилання за `id`) | Покрито NFR, перевірка: тест запису журналу |
| **NFR-11** | Ubiquitous | Адаптивний інтерфейс від 360 px | `UC_Search`, `UC_CreateOrder`, `UC_Review`, `UC_ManageStore`, `UC_ManageProduct` | Не стосується | Покрито NFR, перевірка: огляд у браузері |
| **NFR-12** | Ubiquitous | 500 одночасних сесій покупців без порушення NFR-05, NFR-06 | `UC_Search`, `UC_CreateOrder`, `UC_Auth` | `PRODUCT`, `ORDER` | Покрито NFR, перевірка: навантажувальний тест |
| **NFR-13** | Ubiquitous | Додавання `PRODUCT` ≤ 3 хв для 90% продавців | `UC_ManageProduct`, `UC_ManageStore` | `PRODUCT`, `STORE` | Покрито NFR, перевірка: юзабіліті-тест із продавцями |
| **NFR-14** | Ubiquitous | Оформлення замовлення ≤ 5 кроків від сторінки товару | `UC_Search`, `UC_CreateOrder` | `ORDER`, `ORDER_ITEM`, `PRODUCT` | Покрито NFR, перевірка: юзабіліті-тест із покупцями |
| **NFR-15** | Ubiquitous | Доступність за WCAG 2.1 AA | `UC_Auth`, `UC_Search`, `UC_SearchCat`, `UC_CreateOrder`, `UC_Review`, `UC_ManageStore`, `UC_ManageProduct` | Не стосується | Покрито NFR, перевірка: автоматичний аудит і ручна перевірка |
| **NFR-16** | Ubiquitous | Український інтерфейс і гривня (UAH) | `UC_Auth`, `UC_Search`, `UC_SearchCat`, `UC_CreateOrder`, `UC_Review`, `UC_ManageStore`, `UC_ManageProduct` | `PRODUCT`, `ORDER` (валюта) | Покрито NFR, перевірка: огляд інтерфейсу |
| **NFR-17** | Ubiquitous | Робота в останніх двох версіях Chrome, Firefox, Safari, Edge | `UC_Auth`, `UC_Search`, `UC_CreateOrder`, `UC_Review`, `UC_ManageStore`, `UC_ManageProduct` | Не стосується | Покрито NFR, перевірка: крос-браузерне тестування |

> **Примітка щодо N:M трасування:**
> Вимога **EARS-02** покривається двома прецедентами (`UC_CreateOrder` та `UC_Snapshot`). Прецедент `UC_CreateOrder` бере участь у виконанні вимог **EARS-02**, **EARS-03**, **EARS-04** та **EARS-05**. Вимога **EARS-12** пов'язана з `UC_ExpireOrder` і `UC_Pay` (оплата знімає автоскасування). Вимога **NFR-07** пов'язана з чотирма прецедентами.

## 2. Перевірка покриття Use Cases (зворотний напрям)

Кожен прецедент має щонайменше одну вимогу; жоден не залишився без трасування.

| Use Case | Вимоги EARS | Вимоги NFR |
| :--- | :--- | :--- |
| `UC_Auth` | EARS-11 | NFR-01, NFR-02, NFR-03, NFR-04, NFR-10, NFR-11, NFR-12, NFR-15, NFR-16, NFR-17 |
| `UC_Search` | EARS-01 | NFR-02, NFR-05, NFR-11, NFR-12, NFR-14, NFR-15, NFR-16, NFR-17 |
| `UC_SearchCat` | EARS-01 | NFR-05, NFR-15, NFR-16 |
| `UC_CreateOrder` | EARS-02, EARS-03, EARS-04, EARS-05 | NFR-02, NFR-06, NFR-07, NFR-08, NFR-10, NFR-11, NFR-12, NFR-14, NFR-15, NFR-16, NFR-17 |
| `UC_Snapshot` | EARS-02 | NFR-06, NFR-08 |
| `UC_Pay` | EARS-04, EARS-05, EARS-12, EARS-13 | NFR-02, NFR-07, NFR-10 |
| `UC_ExpireOrder` | EARS-12 | NFR-07, NFR-10 |
| `UC_Review` | EARS-06 | NFR-04, NFR-11, NFR-15, NFR-16, NFR-17 |
| `UC_ManageStore` | EARS-08, EARS-10 | NFR-02, NFR-11, NFR-13, NFR-15, NFR-16, NFR-17 |
| `UC_ManageProduct` | EARS-09, EARS-10 | NFR-07, NFR-10, NFR-11, NFR-13, NFR-15, NFR-16, NFR-17 |

**Непокриті вимоги:** немає. EARS-07 — Out of Scope (розд. 4). NFR-01..NFR-17 трасуються до UC і сутностей (розд. 1), але перевіряються не сценаріями Gherkin, а методами, зазначеними в колонці «Стан покриття». Пороги — у `docs/adr/0009-nfr-thresholds.md`.

## 3. Передумови та постумови Use Cases

### UC_CreateOrder (Оформити замовлення)
- **Preconditions:** Покупець `Customer` авторизований у системі; у кошику є принаймні 1 товар із `stock_quantity > 0`.
- **Postconditions:** Створено запис `ORDER` зі статусом `Pending`; створено позиції `ORDER_ITEM` зі знімком ціни `unit_price`; зменшено доступний `stock_quantity` у `PRODUCT`.

### UC_Pay (Оплатити через шлюз)
- **Preconditions:** Існує замовлення `ORDER` у статусі `Pending` або `Unpaid`. Для `Unpaid` перед оплатою виконується повторний резерв (EARS-13); якщо товару недостатньо, оплата відхиляється.
- **Postconditions:** Статус замовлення змінюється на `Paid` у разі успіху, або скасовується резерв товарів у разі відмови транзакції.

### UC_ExpireOrder (Автоскасування неоплаченого замовлення)
- **Preconditions:** Існує `ORDER` у статусі `Pending`, створений понад 24 години тому, `payment_status` ≠ `approved`.
- **Postconditions:** `ORDER.status` = `Cancelled`; `PRODUCT.stock_quantity` збільшено на `ORDER_ITEM.quantity`.

## 4. Обґрунтування меж системи (Out of Scope)
Вимога **EARS-07** (міжнародне розмитнення та розрахунок митних зборів) свідомо винесена за межі поточного проєкту. Платформа орієнтована на внутрішній ринок електронної комерції та спирається виключно на базові сутності доставки та оплати.
