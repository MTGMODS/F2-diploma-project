# Thesis Proposal

**Кваліфікаційний проєкт бакалавра** · Спеціальність F2 «Інженерія програмного
забезпечення» (попередній шифр 121)

**Автор:** Маргер Богдан Євгенович ·
**Науковий керівник:** Дмитращук Кристина Миколаївна ·
**Статус теми:** попередня, очікує змістовної верифікації керівником.

## 1. Попередня назва

**Українською:** Розроблення мікросервісної платформи для розповсюдження,
монетизації та аналізу використання freemium-продукту.

**English:** Development of a Microservice Platform for Freemium Product
Distribution, Monetization, and Usage Analytics.

## 2. Актуальність і проблема

Незалежному розробнику freemium-продукту потрібна узгоджена система для
представлення продукту, розповсюдження безплатної версії, пропонування платної
підписки, активації доступу та підтримки користувачів. Окремі вебсторінки,
канали купівлі, ліцензійні ключі й файли без спільного обліку ускладнюють
шлях користувача від ознайомлення з продуктом до його використання.
Розробнику також потрібні достовірні дані про запуски продукту та результати
продажів, а користувачам — зрозумілий стан підписки й пристроїв у кабінеті.

Інженерна задача полягає в узгодженій роботі сайту, завантажуваного клієнта,
сервісів облікових записів, ліцензування, розповсюдження VIP-файлу,
телеметрії та інтеграцій із зовнішніми каналами купівлі. Платформа показує
агреговану публічну аналітику використання й продажів. Телеметрія зберігає
ідентифікатор пристрою (HWID), тому в роботі потрібно розмежувати сирі дані,
адміністративний доступ і поля публічних агрегатів, не стверджуючи повної
анонімності.

## 3. Ідея та архітектура

Робота спирається на чинну платформу MTG MODS для завантажуваного хелпера з
безплатною версією та платною підпискою MTGVIP. Публічний сайт представляє
хелпер, пропонує безплатне завантаження для ПК і телефона з інструкціями,
показує тарифи та зовнішні способи придбання, публікує агреговану аналітику
використання й продажів. Особистий кабінет дає змогу активувати ключ,
переглядати підписку, завантажувати VIP-файл і керувати пристроями. Розділ
акцій описує додаткові способи отримання підписки. Коренева сторінка сайту
представляє автора та екосистему; це супровідна сторінка, а не окремий предмет
кваліфікаційної роботи.

Поточна система має чотири API-сервіси:
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

Термін «micro-SaaS» характеризує масштаб і модель незалежного продукту, а
«мікросервісна» — архітектуру реалізації; ці поняття не є взаємозамінними.
Завантажуваний хелпер і вебплатформа працюють разом, тому продукт не слід
описувати як суто браузерний SaaS. Платформа показує платні пропозиції,
спрямовує до зовнішніх каналів купівлі та обліковує суму й канал операції,
але сама не виконує банківський еквайринг. Telegram Stars проходить перевірку
тарифу до підтвердження оплати; інші канали підтверджуються поза платформою
й можуть потребувати ручної видачі ключа.

## 4. Основні функції

1. **Представлення й безплатне розповсюдження.** Сторінка хелпера містить
   опис, скриншоти та перехід до завантаження Free-версії; для ПК і телефона
   доступні файли та інструкції зі встановлення.
2. **Тарифи й канали придбання.** Сторінка MTGVIP показує строки підписки,
   переваги та зовнішні способи купівлі; ключ, отриманий через відповідний
   канал, активується в кабінеті. Платформа не виконує банківський еквайринг.
3. **Обліковий запис і кабінет.** Вхід через Telegram і Discord, підтримка
   Telegram Mini App, прив’язування облікових записів і перегляд стану
   підписки, ключа та історії доступу.
4. **Платний доступ і пристрої.** Генерування й активація ліцензійних ключів,
   контроль строку підписки, прив’язування HWID за тарифною квотою та
   обмежене самостійне скидання пристроїв.
5. **Видача VIP-версії.** Після перевірки доступу сервіс ліцензування надсилає
   запит через RabbitMQ; сервіс розповсюдження формує тимчасовий файл із датою
   завершення доступу й повертає посилання для завантаження.
6. **Облік використання та відкрита аналітика.** API приймає події запуску;
   вебсайт показує агреговані показники, зокрема динаміку, сервери, фракції,
   пристрої та версії. Сирі HWID не належать до публічної відповіді.
7. **Аналітика продажів та інтеграції.** Публічна сторінка показує агреговані
   продажі, канали й тарифи; адміністративний API та Telegram/Discord-боти
   підтримують видачу ключів, перевірку VIP-доступу й супровід користувачів.

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

**Development of a Microservice Platform for Freemium Product Distribution,
Monetization, and Usage Analytics.**

## 2. Relevance and problem statement

An independent freemium product developer needs a coherent system for
presenting the product, distributing its free edition, offering a paid
subscription, activating access, and supporting users. Disconnected web pages,
purchase channels, licence keys, and downloadable files complicate the user
journey from discovery to use. The developer also needs reliable usage and
sales data, while users need a clear view of their subscription and devices.

The engineering problem is coordinating the website, downloadable client,
identity, licensing, VIP-file distribution, telemetry services, and external
purchase-channel integrations. The platform exposes aggregate public usage and
sales analytics. The telemetry database stores device identifiers (HWIDs), so
the work must distinguish raw records, administrative access, and fields
returned in public aggregates without claiming complete anonymity.

## 3. System concept and architecture

The project is based on the operating MTG MODS platform for a downloadable
helper with a free edition and a paid MTGVIP subscription. The public website
presents the helper, offers free PC and mobile downloads with installation
guides, lists paid plans and external purchase options, and publishes aggregate
usage and sales analytics. In the personal dashboard, users can activate a key,
view their subscription, download the VIP file, and manage devices. The
promotions section describes additional ways to receive a subscription. The
website's root page introduces the creator and ecosystem; it is supporting
context, not a separate subject of the thesis.

The current system contains four API services: user identity,
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

“Micro-SaaS” describes the independent product's scale and business model;
“microservice” describes its implementation architecture. These terms are not
interchangeable. Because a downloadable helper works alongside the web
platform, the product should not be described as browser-only SaaS. The
platform lists paid offers, routes users to external purchase channels, and
records transaction amounts and channels, but does not process bank-card
payments itself. Telegram Stars payments are checked against the current
tariff before confirmation; other channels are confirmed outside the platform
and may require manual key issuance.

## 4. Core features

1. **Product presentation and free distribution.** The helper page offers a
   description, screenshots, and a Free download; PC and mobile files come
   with installation guides.
2. **Plans and purchase channels.** The MTGVIP page lists subscription terms,
   benefits, and external purchase methods. A key obtained through the chosen
   channel can be activated in the dashboard; the platform does not perform
   bank-card acquiring.
3. **Accounts and dashboard.** Telegram and Discord sign-in, Telegram Mini App
   support, linked accounts, and views of subscription status, key, and access
   history.
4. **Paid access and devices.** Licence-key generation and activation,
   subscription expiry, tariff-based HWID quotas, and limited self-service
   device resets.
5. **VIP delivery.** After access verification, the licensing service requests
   a temporary file through RabbitMQ; the distribution service adds the access
   expiry date and returns a download link.
6. **Usage records and open analytics.** The API receives launch events; the
   website displays aggregate trends by server, faction, device, and version.
   Raw HWIDs are not included in the public response.
7. **Sales analytics and integrations.** The public website displays aggregate
   sales, channels, and plans. Administrative APIs and Telegram/Discord bots
   support key issuance, VIP checks, and user support.

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
