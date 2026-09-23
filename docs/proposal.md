# Thesis Proposal

**Кваліфікаційний проєкт бакалавра** · Спеціальність F2 «Інженерія програмного
забезпечення» (попередній шифр 121)

**Автор:** Маргер Богдан Євгенович ·
**Науковий керівник:** Дмитращук Кристина Миколаївна ·
**Статус теми:** попередня, очікує змістовної верифікації керівником.

## 1. Попередня назва

**Українською:** Розроблення мікросервісної платформи керування ліцензіями та
розповсюдження цифрового програмного продукту.

**English:** Design and Implementation of a Microservice Platform for Digital
Product Licensing and Distribution.

## 2. Актуальність і проблема

Незалежному постачальнику цифрового програмного продукту потрібні засоби
керування доступом, строком дії ліцензії, прив’язаними пристроями,
розповсюдженням файлів та обліком використання. Ручне надання доступу
ускладнює підтримку актуального стану ліцензій, контроль повторного
використання ключів і отримання узагальнених даних про роботу продукту.

Інженерна задача полягає в узгодженій роботі кількох клієнтів і підсистем:
вебзастосунку, клієнтського програмного продукту, API, фонового оброблення та
інтеграцій із платформами обміну повідомленнями. Система також приймає події
використання й надає агреговану публічну статистику. Телеметрія зберігає
ідентифікатор пристрою (HWID), тому публічну статистику не слід описувати як
гарантовано анонімну: у роботі потрібно чітко розмежувати сирі дані,
адміністративний доступ і поля публічних агрегатів.

## 3. Ідея та архітектура

Робота спирається на production-платформу керування ліцензіями та
розповсюдження цифрового продукту. Поточна система має чотири API-сервіси:
ідентичності користувачів, ліцензування, телеметрії та розповсюдження. Сервіси
ідентичності, ліцензування й телеметрії мають окремі бази PostgreSQL;
розповсюдження обробляє тимчасові файли без власної бази даних. RabbitMQ
використовується для асинхронних команд і запиту-відповіді під час формування
файлу. Вебклієнт взаємодіє з API через nginx; контейнери запускаються через
Docker Compose. Telegram і Discord боти є зовнішніми інтеграційними
адаптерами.

Архітектурний напрям — розподілена мікросервісна система з розподілом
відповідальностей за доменами, окремими сховищами для стану трьох сервісів і
асинхронною взаємодією через RabbitMQ. Це описує наявний код; подальші
архітектурні зміни будуть позначені окремо від уже реалізованих можливостей.

Система обліковує суму й канал операції, але сама не виконує банківський
еквайринг. Telegram Stars проходить перевірку тарифу до підтвердження оплати;
інші канали підтверджуються поза платформою й можуть потребувати ручної
видачі ключа.

## 4. Основні функції

1. **Автентифікація та обліковий запис.** Вхід через Telegram і Discord,
   підтримка Telegram Mini App, прив’язування соціальних облікових записів і
   короткоживучі access-токени з поновлюваними refresh-сесіями.
2. **Життєвий цикл ліцензії.** Генерування ключів, активація, обмеження до
   однієї активної ліцензії на користувача, примусова заміна та фонова
   обробка завершення строку дії.
3. **Керування пристроями.** Прив’язування HWID до ліцензії за тарифною
   квотою, обмеження самостійного скидання пристроїв і адміністративне
   керування прив’язками.
4. **Формування та видача файлу.** Сервіс ліцензування надсилає запит через
   RabbitMQ, сервіс розповсюдження формує файл із датою завершення доступу й
   повертає посилання. Файл видаляється після передавання; фонове очищення
   прибирає залишкові файли.
5. **Телеметрія використання.** API перевіряє версію продукту, режим, сервер,
   пристрій і HWID та зберігає події запуску для подальших агрегатів.
6. **Публічна аналітика.** Вебклієнт показує агреговані дані використання й
   продажів. Сира телеметрія та HWID не є частиною публічної відповіді;
   твердження про повне знеособлення потребує окремого аналізу ризику
   повторної ідентифікації.
7. **Адміністрування та боти.** Адміністративний API керує користувачами й
   ліцензіями; адаптери Telegram і Discord перевіряють VIP-доступ і
   обробляють команди, пов’язані зі статусом ліцензії.

## 5. Технологічний стек

| Категорія | Поточні технології |
|---|---|
| Мови | Python 3.12, TypeScript, SQL, Bash |
| Серверні засоби | FastAPI, SQLAlchemy 2, Pydantic, HTTPX, aio-pika |
| Вебклієнт | React 19, Vite, TanStack Query, Zustand, React Router, Tailwind CSS, Recharts, i18next |
| Інтеграції | python-telegram-bot, discord.py |
| Бази даних | PostgreSQL 16: окремі екземпляри для user, license та usage сервісів |
| Черга повідомлень | RabbitMQ 3.13, AMQP |
| Розгортання | Docker, Docker Compose, nginx |

## 6. Обсяг кваліфікаційної роботи

Production-система вже працює; дипломна робота описує й обґрунтовує її
інженерні рішення, перевіряє визначені властивості та фіксує обсяг подальших
змін. Автоматизовані тести, CI, міграції баз даних, навантажувальне тестування
та додаткові засоби спостережуваності розглядатимуться як запланована робота
лише після остаточного визначення обсягу. Їх не подано як наявні можливості.

---

# Thesis Proposal (English)

**Bachelor’s qualification project** · F2 Software Engineering (previous code
121)

**Author:** Bohdan Marher ·
**Supervisor:** Dmytrashchuk Krystyna Mykolaivna · **Topic status:**
preliminary; substantive supervisor review is pending.

## 1. Preliminary title

**Design and Implementation of a Microservice Platform for Digital Product
Licensing and Distribution.**

## 2. Relevance and problem statement

An independent digital product vendor needs ways to manage access, licence
duration, bound devices, file distribution, and usage records. Manual access
provisioning makes it harder to keep licence state current, prevent key reuse,
and obtain aggregate information about product usage.

The engineering problem is coordinating several clients and subsystems: a web
application, a client-side software product, APIs, background processing, and
messaging-platform integrations. The system also ingests usage events and
exposes aggregate public statistics. The telemetry database stores device
identifiers (HWIDs), so the work must distinguish raw records, administrative
access, and fields returned in public aggregates instead of claiming guaranteed
anonymity.

## 3. System concept and architecture

The project is based on a production platform for licensing and distributing a
digital product. It currently contains four API services: user identity,
licensing, usage telemetry, and distribution. The identity, licensing, and
telemetry services each have a separate PostgreSQL database; the distribution
service processes temporary files without its own database. RabbitMQ carries
asynchronous commands and request-reply messages used during file generation.
The web client reaches the APIs through nginx; Docker Compose runs the
containers. Telegram and Discord bots act as external integration adapters.

The architectural direction is a distributed microservice system with
domain-oriented responsibilities, separate stores for the state of three
services, and asynchronous communication through RabbitMQ. This describes the
existing code; future changes will be identified separately from implemented
capabilities.

The platform records transaction amounts and channels but does not process
bank-card payments itself. Telegram Stars payments are checked against the
current tariff before confirmation; other channels are confirmed outside the
platform and may require manual key issuance.

## 4. Core features

1. **Authentication and accounts.** Sign-in through Telegram and Discord,
   Telegram Mini App support, social-account linking, short-lived access tokens,
   and renewable refresh sessions.
2. **Licence lifecycle.** Key generation and activation, a single-active
   licence constraint per user, forced replacement, and background expiry
   handling.
3. **Device management.** Binding HWIDs within a tariff quota, limiting
   self-service device resets, and administrative management of bindings.
4. **File generation and delivery.** The licensing service sends a RabbitMQ
   request; the distribution service creates a file containing an access expiry
   date and returns a link. The file is removed after transfer, with a
   background worker cleaning up leftovers.
5. **Usage telemetry.** The API validates product version, mode, server, device,
   and HWID, then stores launch events for aggregation.
6. **Public analytics.** The web client displays aggregate usage and sales
   figures. Raw telemetry and HWIDs are not part of the public response; any
   claim of complete de-identification requires a separate re-identification
   risk analysis.
7. **Administration and bots.** The administrative API manages users and
   licences; Telegram and Discord adapters check VIP access and handle
   licence-related commands.

## 5. Technology stack

| Category | Current technologies |
|---|---|
| Languages | Python 3.12, TypeScript, SQL, Bash |
| Backend | FastAPI, SQLAlchemy 2, Pydantic, HTTPX, aio-pika |
| Web client | React 19, Vite, TanStack Query, Zustand, React Router, Tailwind CSS, Recharts, i18next |
| Integrations | python-telegram-bot, discord.py |
| Databases | PostgreSQL 16: separate instances for user, license, and usage services |
| Message broker | RabbitMQ 3.13, AMQP |
| Deployment | Docker, Docker Compose, nginx |

## 6. Project scope

The production system is already in operation. The thesis will describe and
justify its engineering decisions, verify selected properties, and define any
further work. Automated tests, CI, database migrations, load testing, and
additional observability will be treated as planned work only after the final
scope has been agreed; they are not presented as existing capabilities.
