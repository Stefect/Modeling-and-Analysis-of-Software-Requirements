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
| **EARS-07** | Optional | Розрахунок міжнародної митної вартості та зборів (WHERE доставка за межі ринку) | *Відсутній* | *Відсутні* | **Out of Scope** |

> **Примітка щодо N:M трасування:**
> Вимога **EARS-02** покривається двома прецедентами (`UC_CreateOrder` та `UC_Snapshot`). Прецедент `UC_CreateOrder` бере участь у виконанні вимог **EARS-02**, **EARS-03**, **EARS-04** та **EARS-05**. Вимога **EARS-12** пов'язана з `UC_ExpireOrder` і `UC_Pay` (оплата знімає автоскасування).

## 2. Перевірка покриття Use Cases (зворотний напрям)

Кожен прецедент має щонайменше одну вимогу; жоден не залишився без трасування.

| Use Case | Вимоги EARS |
| :--- | :--- |
| `UC_Auth` | EARS-11 |
| `UC_Search` | EARS-01 |
| `UC_SearchCat` | EARS-01 |
| `UC_CreateOrder` | EARS-02, EARS-03, EARS-04, EARS-05 |
| `UC_Snapshot` | EARS-02 |
| `UC_Pay` | EARS-04, EARS-05, EARS-12 |
| `UC_ExpireOrder` | EARS-12 |
| `UC_Review` | EARS-06 |
| `UC_ManageStore` | EARS-08, EARS-10 |
| `UC_ManageProduct` | EARS-09, EARS-10 |

**Непокриті вимоги:** немає, крім EARS-07 (Out of Scope, розд. 3).

## 3. Передумови та постумови Use Cases

### UC_CreateOrder (Оформити замовлення)
- **Preconditions:** Покупець `Customer` авторизований у системі; у кошику є принаймні 1 товар із `stock_quantity > 0`.
- **Postconditions:** Створено запис `ORDER` зі статусом `Pending`; створено позиції `ORDER_ITEM` зі знімком ціни `unit_price`; зменшено доступний `stock_quantity` у `PRODUCT`.

### UC_Pay (Оплатити через шлюз)
- **Preconditions:** Існує замовлення `ORDER` у статусі `Pending` або `Unpaid`.
- **Postconditions:** Статус замовлення змінюється на `Paid` у разі успіху, або скасовується резерв товарів у разі відмови транзакції.

### UC_ExpireOrder (Автоскасування неоплаченого замовлення)
- **Preconditions:** Існує `ORDER` у статусі `Pending`, створений понад 24 години тому, `payment_status` ≠ `approved`.
- **Postconditions:** `ORDER.status` = `Cancelled`; `PRODUCT.stock_quantity` збільшено на `ORDER_ITEM.quantity`.

## 4. Обґрунтування меж системи (Out of Scope)
Вимога **EARS-07** (міжнародне розмитнення та розрахунок митних зборів) свідомо винесена за межі поточного проєкту. Платформа орієнтована на внутрішній ринок електронної комерції та спирається виключно на базові сутності доставки та оплати.
