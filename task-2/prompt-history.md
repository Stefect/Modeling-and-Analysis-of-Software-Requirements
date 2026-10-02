# Журнал взаємодії з AI (Prompt History) — Завдання 2

## Ітерація 1: Первинна генерація вимог та Use Case діаграми

**Промт:**
> "На основі сутностей з Завдання 1 (USER, STORE, PRODUCT, ORDER, ORDER_ITEM, REVIEW) сформуй для мультивендорного маркетплейсу:
> 1. Набір функціональних вимог у форматі EARS для покупця, продавця та платіжного шлюзу.
> 2. User Story та Gherkin-сценарій.
> 3. Use Case діаграму у Mermaid з акторами та зв'язками."

**Отриманий результат від AI:**

### EARS-вимоги:
- EARS-01: Система надає можливість пошуку товарів за назвою та категорією.
- EARS-02: WHEN покупець тисне "Оформити замовлення", система SHALL створювати замовлення ORDER.
- EARS-03: WHILE замовлення перебуває в статусі "Pending", система SHALL резервувати товари PRODUCT.

### Use Case Diagram (Mermaid Чернетка):
flowchart TD
    Customer([Customer])
    Seller([Seller])
    PaymentGateway([Payment Gateway])

    subgraph Marketplace ["Маркетплейс"]
        UC1(Шукати товари)
        UC2(Оформити замовлення)
        UC3(Оплатити замовлення)
        UC4(Керувати каталогом)
    end

    Customer --> UC1
    Customer --> UC2
    Customer --> UC3
    Seller --> UC4
    UC3 --> PaymentGateway
    UC2 .->|include| UC3

---

## Ітерація 2: Поглиблення вимог EARS, корекція Use Cases та генералізація акторів

**Промт:**
> "Доопрацюй вимоги та Use Case діаграму Завдання 2 з урахуванням деталей архітектури:
> 1. Додай абстрактного актора User та реалізуй успадкування (Customer, Seller -> User).
> 2. Виправ помилку в діаграмі: зміни зв'язок між 'Оформити замовлення' та 'Оплатити замовлення' на extend.
> 3. Додай обов'язковий include-сценарій 'Фіксація ціни unit_price в ORDER_ITEM' (Price Snapshot з Лабораторної 1).
> 4. Розшир EARS-вимоги для покриття відгуків REVIEW, вибору категорій CATEGORY та помилок відмови платежу."

**Отриманий результат від AI:**

### Оновлені EARS-вимоги:
- EARS-01 (Ubiquitous): Система SHALL надавати можливість пошуку та фільтрації товарів PRODUCT за ієрархічними категоріями CATEGORY.
- EARS-02 (Event-driven): WHEN покупець Customer підтверджує замовлення, система SHALL створювати запис ORDER та фіксувати знімок ціни unit_price у сутності ORDER_ITEM.
- EARS-03 (State-driven): WHILE замовлення перебуває у статусі Pending, система SHALL блокувати відповідну кількість товарних залишків stock_quantity.
- EARS-04 (Optional): WHERE покупець обирає онлайн-оплату, система SHALL ініціювати транзакцію через Payment Gateway.
- EARS-05 (Unwanted Behavior): IF Payment Gateway повертає статус відмови транзакції, THEN система SHALL скасовувати резерв товарів та залишати ORDER у статусі Unpaid.
- EARS-06 (Event-driven): WHEN покупець Customer залишає відгук REVIEW, система SHALL зберігати оцінку та коментар з прив'язкою до PRODUCT.

### User Stories & Gherkin:
- US-01: Як Customer, я хочу оформлювати замовлення з фіксацією ціни на момент купівлі, щоб уникнути зміни вартості товарів у разі оновлення прайсу продавцем.
  - Gherkin:
    - Given покупець додав товари з каталогу у кошик
    - When покупець підтверджує оформлення замовлення
    - Then система створює новий запис ORDER
    - And для кожного товару фіксується поточна ціна у ORDER_ITEM.unit_price

### Оновлена Use Case Diagram (Mermaid):
flowchart TD
    User([User])
    Customer([Customer])
    Seller([Seller])
    PaymentGateway([Payment Gateway])

    Customer --|> User
    Seller --|> User

    subgraph Marketplace ["Мультивендорний Маркетплейс"]
        UC_Auth(Авторизуватися в системі)
        UC_Search(Шукати товари)
        UC_SearchCat(Фільтрувати за CATEGORY)
        UC_CreateOrder(Оформити замовлення ORDER)
        UC_Snapshot(Зафіксувати ціну unit_price)
        UC_Pay(Оплатити через шлюз)
        UC_Review(Залишити відгук REVIEW)
        UC_ManageStore(Керувати магазином STORE)
        UC_ManageProduct(Управляти товарами PRODUCT)
    end

    User --> UC_Auth
    Customer --> UC_Search
    Customer --> UC_CreateOrder
    Customer --> UC_Review
    
    Seller --> UC_ManageStore
    Seller --> UC_ManageProduct

    UC_SearchCat --|> UC_Search

    UC_CreateOrder .->|"<<include>>"| UC_Snapshot
    UC_Pay .->|"<<extend>>"| UC_CreateOrder

    UC_Pay --> PaymentGateway