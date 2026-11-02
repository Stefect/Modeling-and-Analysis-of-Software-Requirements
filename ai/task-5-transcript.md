# Транскрипт сесії — Завдання 5 (Component Diagram)

## Підсумок
Сесія присвячена моделюванню архітектури мультивендорного маркетплейсу. AI використано для аналізу попередніх артефактів, генерації PlantUML-діаграми та Mermaid-версії. Студент перевірив узгодженість із ERD, Use Cases, EARS/NFR та Activity-діаграмою, виправив 3 розбіжності й зафіксував рішення в ADR-0016.

## Артефакти
* `task-5/spec.md` — специфікація архітектури.
* `task-5/component.puml` — еталонна UML Component Diagram.
* `task-5/component.md` — Mermaid-версія для GitHub.
* `task-5/DEFENSE.md` — захист за шаблоном DoD §4.
* `docs/adr/0016-component-architecture-marketplace.md` — ADR.

## Рішення студента (ключові)
1. Модульний моноліт замість мікросервісів (транзакційність, обсяг курсу).
2. `Outbox Relay Worker` і `Order Expiry Scheduler` — окремі компоненти.
3. `Identity Provider` — опційна інтеграція.