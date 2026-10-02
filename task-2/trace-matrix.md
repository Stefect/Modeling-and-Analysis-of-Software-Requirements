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
| **EARS-07** | Optional | Розрахунок міжнародної митної вартості та логістики | *Відсутній* | *Відсутні* | **Out of Scope** |

> **Примітка щодо $N:M$ трасування:** 
> Вимога **EARS-02** покривається одразу двома прецедентами (`UC_CreateOrder` та `UC_Snapshot`), а прецедент `UC_CreateOrder` бере участь у виконанні вимог **EARS-02**, **EARS-03**, **EARS-04** та **EARS-05**.

## 2. Передумови та постумови Use Cases

### UC_CreateOrder (Оформити замовлення)
- **Preconditions:** Покупець `Customer` авторизований у системі; у кошику є принаймні 1 товар із `stock_quantity > 0`.
- **Postconditions:** Створено запис `ORDER` зі статусом `Pending`; створено позиції `ORDER_ITEM` зі знімком ціни `unit_price`; зменшено доступний `stock_quantity` у `PRODUCT`.

### UC_Pay (Оплатити через шлюз)
- **Preconditions:** Існує замовлення `ORDER` у статусі `Pending` або `Unpaid`.
- **Postconditions:** Статус замовлення змінюється на `Paid` у разі успіху, або скасовується резерв товарів у разі відмови транзакції.

## 3. Обґрунтування меж системи (Out of Scope)
Вимога **EARS-07** (міжнародне розмитнення та розрахунок митних зборів) свідомо винесена за межі поточного проєкту. Платформа орієнтована на внутрішній ринок електронної комерції та спирається виключно на базові сутності доставки та оплати.