# Thesis Proposal

**Кваліфікаційний проєкт бакалавра** · Спеціальність F2 «Інженерія програмного
забезпечення» (попередній шифр 121)

**Автор:** Маргер Богдан Євгенович ·
**Науковий керівник:** Дмитращук Кристина Миколаївна ·
**Статус теми:** попередня, очікує змістовної верифікації керівником.

## 1. Попередня назва

**Українською:** Розроблення мікросервісної платформи для розповсюдження,
керування платним доступом та аналізу використання наявного цифрового продукту.

**English:** Development of a Microservice Platform for Distribution, Paid
Access Management, and Usage Analytics of an Existing Digital Product.

## 2. Актуальність і проблема

Предметна область проєкту — процеси розповсюдження, ліцензування,
керування платним доступом та аналізу використання вже готових цифрових
програмних продуктів. У цьому кейсі зовнішнім продуктом є завантажуваний
програмний помічник (хелпер) для гравців San Andreas Multiplayer (SA-MP),
зокрема спільнот Arizona RP і Rodina RP. Він існував понад два роки до
створення нової платформи, має безплатну версію та платні можливості за
моделлю freemium. Розроблення самого помічника **не входить** до цієї
кваліфікаційної роботи.

У постановці задачі власник готового продукту виступає замовником платформи,
яка забезпечує його представлення, розповсюдження, керування підпискою,
активацію доступу й підтримку користувачів. До появи програмної системи
доступ продавався вручну через чати, а пізніше обліковувався монолітним
застосунком. Саме **нинішня мікросервісна платформа** почала формуватися на
Етапі 3 — як університетський репозиторій мікросервісів, що використовував
предметну область продукту й попередній досвід. Це не було прямим перенесенням
коду моноліту. На Етапі 4 цю навчальну систему еволюційно доведено до
production-платформи з вебсайтом, а на Етапі 5 об'єднано застосунок, вебклієнт
і ботів у спільний монорепозиторій. Кваліфікаційний проєкт продовжує цю
еволюцію від Етапу 3; попередні ручний і монолітний етапи є історичним
контекстом і джерелом вимог. Окремі вебсторінки, канали купівлі, ключі та
файли без спільного обліку ускладнювали шлях користувача від ознайомлення до
використання. Власнику потрібні достовірні дані про запуски й продажі, а
користувачам — зрозумілий стан підписки та пристроїв.

Інженерна задача полягає в узгодженій роботі нової платформи з уже наявним
цифровим продуктом: сайту, сервісів облікових записів, ліцензування,
розповсюдження VIP-файлу, телеметрії та інтеграцій із зовнішніми каналами
купівлі. Платформа показує агреговану публічну аналітику використання й
продажів. Телеметрія зберігає
ідентифікатор пристрою (HWID), тому в роботі потрібно розмежувати сирі дані,
адміністративний доступ і поля публічних агрегатів, не стверджуючи повної
анонімності.

## 3. Ідея та архітектура

Об'єкт розроблення й аналізу — мікросервісна платформа MTG MODS, започаткована
на Етапі 3 та розвинена на Етапах 4–5 до чинної production-системи для вже
готового програмного помічника з безплатною версією та платною підпискою
MTGVIP. Публічний сайт представляє продукт, пропонує безплатне завантаження
для ПК і телефона з інструкціями,
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
Еволюційний перехід від навчальної декомпозиції Етапу 3 до production-розгортання
Етапів 4–5 є частиною історії та інженерного контексту кваліфікаційного проєкту.

Термін «micro-SaaS» може описувати модель сервісу навколо готового продукту, а
«мікросервісна» — архітектуру платформи; ці поняття не є взаємозамінними.
Платформа інтегрується з наявним завантажуваним помічником, тому її не слід
видавати за розробку самого помічника. Платформа показує платні пропозиції,
спрямовує до зовнішніх каналів купівлі та обліковує суму й канал операції,
але сама не виконує банківський еквайринг. Telegram Stars проходить перевірку
тарифу до підтвердження оплати; інші канали підтверджуються поза платформою
й можуть потребувати ручної видачі ключа.

## 4. Основні функції

1. **Представлення й безплатне розповсюдження.** Сторінка помічника містить
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
| Архітектурні рішення | Декомпозиція за доменами, окремі БД сервісів, REST API, асинхронний обмін повідомленнями |
| Серверні засоби | FastAPI, SQLAlchemy 2, Pydantic, HTTPX, aio-pika |
| Вебклієнт | React 19, Vite, TanStack Query, Zustand, React Router, Tailwind CSS, Recharts, i18next |
| Інтеграції | python-telegram-bot, discord.py |
| Бази даних | PostgreSQL 16: окремі екземпляри для user, license та usage сервісів |
| Черга повідомлень | RabbitMQ 3.13, AMQP |
| Розгортання | Docker, Docker Compose, nginx |

## 6. Обсяг кваліфікаційної роботи

Кваліфікаційний проєкт описує та продовжує еволюцію платформи, розпочату на
Етапі 3 з університетського репозиторію мікросервісів і доведену на Етапах 4–5
до production-системи. Раніше створений SA-MP помічник залишається зовнішнім
продуктом-кейсом; дипломна робота стосується платформи для нього. Вона має
зафіксувати архітектурні рішення, шлях еволюції та перевірити інженерні
властивості на відтворюваних сценаріях.
Для погодження з керівником запропоновано три напрями розвитку: автоматизовані
тести та CI для критичних сценаріїв ліцензування; аналіз і посилення надійності
асинхронної видачі VIP-файлу; вимірювання часу відповіді публічної аналітики
під навантаженням. Конкретний обсяг реалізації й критерії перевірки ще мають
бути затверджені. Тести, CI та навантажувальні перевірки не подано як уже
реалізовані можливості.

---

# Thesis Proposal (English)

**Bachelor’s qualification project** · F2 Software Engineering (previous code
121)

**Author:** Bohdan Marher ·
**Supervisor:** Dmytrashchuk Krystyna Mykolaivna · **Topic status:**
preliminary; substantive supervisor review is pending.

## 1. Preliminary title

**Development of a Microservice Platform for Distribution, Paid Access
Management, and Usage Analytics of an Existing Digital Product.**

## 2. Relevance and problem statement

The domain covers the processes of distributing, licensing, managing paid
access to, and analysing the use of existing digital software products. In
this case, the external product is a downloadable software
assistant (helper) for San Andreas Multiplayer (SA-MP) players, including the
Arizona RP and Rodina RP communities. It existed for more than two years
before the new platform and offers a free edition and paid features under a
freemium model. Developing the assistant itself is **outside the scope** of
this qualification project.

In the project scenario, the owner of the existing product acts as the client
for a platform that presents and distributes the product, manages subscriptions
and access activation, and supports users. Before software-based management,
access was sold manually through chats and later tracked by a monolithic
application. The **current microservice platform** began to take shape in
Stage 3 as a university microservices repository based on the product domain
and earlier experience. It was not a direct code port of the monolith. In
Stage 4, this educational system evolved into a production platform with a
website; in Stage 5, the application, web client, and bots were consolidated
into one monorepository. The qualification project continues this evolution
from Stage 3; the earlier manual and monolithic stages provide historical
context and requirements. Disconnected pages, purchase channels, licence
keys, and files complicated the user journey. The owner needs reliable usage
and sales data, while users need a clear view of subscriptions and devices.

The engineering problem is coordinating the new platform with the existing
digital product through the website, identity, licensing, VIP-file
distribution, telemetry services, and external purchase-channel integrations.
The platform exposes aggregate public usage and sales analytics. The
telemetry database stores device identifiers (HWIDs), so
the work must distinguish raw records, administrative access, and fields
returned in public aggregates without claiming complete anonymity.

## 3. System concept and architecture

The subject of development and analysis is the MTG MODS microservice platform,
started in Stage 3 and evolved in Stages 4–5 into the current production system
for the pre-existing software assistant and its paid MTGVIP subscription. The
public website presents the product and offers free PC and
mobile downloads with installation
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
capabilities. The evolution from the educational decomposition in Stage 3 to
production deployment in Stages 4–5 is part of the qualification project's
history and engineering context.

“Micro-SaaS” may describe the service model around the existing product;
“microservice” describes the platform's implementation architecture. These
terms are not interchangeable. The platform integrates with the pre-existing
downloadable assistant; it does not constitute development of that assistant.
The platform lists paid offers, routes users to external purchase channels, and
records transaction amounts and channels, but does not process bank-card
payments itself. Telegram Stars payments are checked against the current
tariff before confirmation; other channels are confirmed outside the platform
and may require manual key issuance.

## 4. Core features

1. **Product presentation and free distribution.** The assistant page offers a
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
| Architectural decisions | Domain-oriented services, separate service databases, REST APIs, asynchronous messaging |
| Backend | FastAPI, SQLAlchemy 2, Pydantic, HTTPX, aio-pika |
| Web client | React 19, Vite, TanStack Query, Zustand, React Router, Tailwind CSS, Recharts, i18next |
| Integrations | python-telegram-bot, discord.py |
| Databases | PostgreSQL 16: separate instances for user, license, and usage services |
| Message broker | RabbitMQ 3.13, AMQP |
| Deployment | Docker, Docker Compose, nginx |

## 6. Project scope

The qualification project documents and continues the platform's evolution,
which began as a university microservices repository in Stage 3 and reached
production in Stages 4–5. The previously developed SA-MP assistant is the
external product case; the thesis addresses the platform built for it. The
work will document architectural decisions, the evolution path, and evaluate
engineering properties using reproducible scenarios.
Three development directions are proposed for supervisor review: automated
tests and CI for critical licensing flows; analysis and improvement of the
reliability of asynchronous VIP-file delivery; and load-based measurement of
public analytics response time. The exact implementation scope and evaluation
criteria still require approval. Tests, CI, and load tests are not presented
as existing capabilities.
