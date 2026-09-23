# Thesis Proposal

**Кваліфікаційний проєкт бакалавра** · Спеціальність F2 «Інженерія програмного забезпечення» (попередній код 121)

**Автор:** Маргер Богдан Євгенович
**Науковий керівник:** Дмитращук Кристина Миколаївна
**Статус теми:** попередня; керівниця підтвердила актуальність і практичну спрямованість теми та рекомендувала зосередитися на обґрунтуванні архітектури, ліцензуванні, підписках, пристроях і зовнішніх інтеграціях. Остаточне формулювання та обсяг роботи уточнюються.

## 1. Попередня назва

**Українською:** Розроблення micro-SaaS-платформи для розповсюдження, ліцензування та аналітики цифрового продукту на основі мікросервісної архітектури.

**English:** Development of a Micro-SaaS Platform for Digital Product Distribution, Licensing, and Analytics Using a Microservices Architecture.

## 2. Актуальність і проблема

Предметна область проєкту — надання вузькоспеціалізованого micro-SaaS-сервісу для управління цифровим продуктом: його розповсюдженням, підписками, ліцензіями, файлами доступу та аналітикою продажів і використання. У проєктному сценарії замовником є власник уже готового цифрового програмного продукту. Для конкретизації вимог розглядається завантажуваний помічник для гравців San Andreas Multiplayer (SA-MP), зокрема спільнот Arizona RP і Rodina RP. Помічник існував до створення нової платформи, має безплатну версію та платні можливості за моделлю freemium. Його розроблення не є метою кваліфікаційного проєкту.

Вихідною системою для визначення вимог є наявний моноліт, який використовувався для обліку продажів і керування доступом до готового продукту. Його початковий стан збережено в новому репозиторії як `v0.0.0`, щоб зафіксувати перелік функцій і правил, які власник продукту просив врахувати в окремій платформі. Серед таких вимог — збереження сумісності ліцензійного ключа та підтримка прив’язування облікових записів через Telegram, Discord або обидва канали. Вихідна система є референсом для вимог, а не предметом розроблення; нова платформа не описується як просте копіювання її коду.

Інженерна проблема полягає у створенні окремої розподіленої платформи, яка узгоджує облікові записи, підписки, ліцензії, перевірки пристроїв, видачу файлів, телеметрію та зовнішні канали придбання. Платформа надає особистий кабінет, розповсюджує безплатну версію, видає платні файли відповідно до чинної підписки та формує відкриті агрегати використання й продажів. Сайт спрямовує користувача до зовнішніх каналів оплати й обліковує отримані дані про операції, але сам не здійснює банківський еквайринг.

## 3. Ідея та архітектура

Ідея роботи — micro-SaaS-платформа для власника цифрового продукту, яка об'єднує його вебпредставлення, розповсюдження файлів, керування платним доступом і пристроями, а також аналітику. Термін **micro-SaaS** тут описує модель вузькоспеціалізованого програмного сервісу для підтримки продукту; **мікросервісна архітектура** описує побудову самої платформи. Це різні характеристики однієї системи.

Продукт-кейс — готовий SA-MP помічник із безплатною версією та підпискою MTGVIP. Сайт представляє його, надає файли й інструкції для ПК та телефона, показує тарифи й переходи до зовнішніх способів придбання, а також публічну агреговану статистику. Особистий кабінет дає змогу входити через Telegram або Discord, активувати ключ, переглядати підписку, отримувати VIP-файл і керувати прив'язаними пристроями. Telegram і Discord боти виступають інтеграційними адаптерами. Головна сторінка автора є супровідною частиною сайту, а не окремим об'єктом розроблення.

Архітектура платформи декомпонує функції облікових записів, ліцензування, використання продукту та розповсюдження файлів між окремими API-сервісами. Обмін із клієнтом і вебзастосунком відбувається через REST API; для частини міжсервісних сценаріїв використовується RabbitMQ. Така побудова дає змогу окремо визначати відповідальність компонентів і керувати пов’язаними даними, зберігаючи спільні бізнес-сценарії продукту.

Поточна production-система містить чотири API-сервіси: user, license, usage і distribution. Сервіси user, license та usage мають окремі PostgreSQL бази; distribution обробляє тимчасові файли без власної бази даних. RabbitMQ забезпечує асинхронні команди та взаємодію запит-відповідь під час видачі файлу. Вебклієнт побудований на React і TypeScript, API проходять через nginx, контейнери запускаються Docker Compose. Це наявна реалізація; запропоновані подальші зміни позначаються окремо.

Основний напрям Додатка 1 до ЛР — розподілені системи та Cloud-Native мікросервісні архітектури: платформа має сервіси з окремими відповідальностями й сховищами даних та асинхронну взаємодію через RabbitMQ. AI & Data Engineering, Application Security & DevSecOps і DevOps & Quality Engineering дотичні до аналітики, захисту й перевірки якості, але не визначають основну архітектурну задачу роботи. Micro-SaaS є вузькою сервісною моделлю, а мікросервіси описують спосіб побудови платформи.

## 4. Функції платформи

1. **Представлення та розповсюдження продукту:** сторінка з описом, зображеннями, безплатними файлами для ПК і телефона та інструкціями.
2. **Тарифи й зовнішні канали придбання:** відображення вартості й умов MTGVIP, перенаправлення до каналів купівлі, облік джерела операції та активація отриманого ключа.
3. **Обліковий запис і кабінет:** вхід через Telegram і Discord, прив'язування облікових записів, перегляд підписки, ключа та історії доступу.
4. **Ліцензування й пристрої:** створення та активація ключів, перевірка строку дії, облік пристроїв за HWID і тарифними квотами.
5. **Захищена видача платного файлу:** перевірка доступу та асинхронне формування тимчасового VIP-файлу із завершенням строку підписки.
6. **Телеметрія та відкрита аналітика використання:** приймання подій запуску й показ агрегованих даних за часом, серверами, фракціями, пристроями та версіями. Сирі HWID не публікуються.
7. **Аналітика продажів та інтеграції:** агреговані показники продажів, тарифів і каналів; адміністративний API та Telegram/Discord боти.

## 5. Технологічний стек

| Категорія | Реалізація |
|---|---|
| Мови | Python 3.12, TypeScript, SQL, Bash |
| Архітектурні патерни та інтерфейси | Доменна декомпозиція на мікросервіси; REST API; асинхронні повідомлення; взаємодія запит-відповідь |
| Backend-фреймворки та бібліотеки | FastAPI, SQLAlchemy 2, Pydantic, HTTPX, aio-pika |
| Web-фреймворки та бібліотеки | React 19, Vite, TanStack Query, Zustand, React Router, Tailwind CSS, Recharts, i18next |
| Інтеграції | python-telegram-bot, discord.py |
| Бази даних | PostgreSQL 16: окремі сховища user, license та usage сервісів |
| Брокер | RabbitMQ 3.13, AMQP |
| DevOps та засоби розгортання | Docker, Docker Compose, nginx |

## 6. Обсяг і напрями інженерної роботи

Проєкт охоплює аналіз вимог, сформованих за попередньою системою та побажаннями власника готового продукту, і дослідження створеної для нього micro-SaaS-платформи. Основний інженерний фокус — обґрунтування мікросервісної декомпозиції та взаємодії компонентів, механізмів ліцензування й підписок, прив’язування облікових записів і пристроїв, видачі файлів, інтеграцій та аналітики.

Для погодження з керівником запропоновано перевірювані подальші задачі: автоматизувати тести критичних сценаріїв ключів, підписок і HWID; оцінити надійність асинхронної видачі файлу при повторних запитах, затримках і відмовах; виміряти продуктивність API аналітики під контрольованим навантаженням; перевірити захист чутливих телеметричних даних і секретів. Це пропозиції для дипломної роботи, а не твердження про вже реалізовані перевірки. Конкретний обсяг і критерії оцінювання мають бути погоджені з керівником.

---

# Thesis Proposal (English)

**Bachelor’s qualification project** · F2 Software Engineering (previous code 121)

**Author:** Bohdan Marher
**Supervisor:** Krystyna Dmytrashchuk
**Topic status:** preliminary; the supervisor has confirmed its relevance and practical value and recommended focusing on architecture, licensing, subscriptions, devices, and external integrations. The final wording and scope remain subject to refinement.

## 1. Preliminary title

**Development of a Micro-SaaS Platform for Digital Product Distribution, Licensing, and Analytics Using a Microservices Architecture.**

## 2. Relevance and problem statement

The subject area is a focused micro-SaaS platform for a completed digital product: its presentation and distribution, subscription and licence management, access files, and sales and usage analytics. In the project scenario, the client is the owner of an existing digital software product. The case used to make the requirements concrete is a downloadable assistant for San Andreas Multiplayer (SA-MP) players, including the Arizona RP and Rodina RP communities. The assistant existed before the new platform and has a free edition and paid MTGVIP features. Developing the assistant itself is outside the scope of this qualification project.

The previous system used as the requirements baseline was a monolith for recording sales and managing access to the completed product. Its initial state was preserved in the new repository as `v0.0.0` to capture the features and rules the product owner asked to account for in a separate platform. These include preserving licence-key compatibility and supporting account linking through Telegram, Discord, or both channels. The previous system is a reference for requirements, not the subject of development; the new platform is not described as a simple copy of its code.

The engineering problem is building a separate distributed platform that coordinates accounts, subscriptions, licences, device checks, file delivery, telemetry, and external purchase channels. The platform provides a user dashboard, free-edition distribution, paid files subject to an active subscription, and public aggregate usage and sales analytics. The website routes users to external payment channels and records transaction information; it does not process bank-card payments itself.

## 3. Concept and architecture

The project develops a micro-SaaS platform for a digital-product owner. It brings together web presentation, file distribution, paid-access and device management, and analytics. **Micro-SaaS** describes the focused service model around the product; **microservices** describe how the platform is architected. These are different properties of the same system.

The product case is an existing SA-MP assistant with a free edition and MTGVIP subscription. The website presents it, provides PC and mobile files and guides, lists plans and external purchase options, and publishes aggregate statistics. The user dashboard supports Telegram or Discord sign-in, key activation, subscription status, VIP-file downloads, and linked-device management. Telegram and Discord bots serve as integration adapters. The creator's home page supports the website but is not a separate development object.

The previous monolithic system is used as a functional reference for the new platform. Its initial state was preserved in the new repository as `v0.0.0` to make the existing feature set available when formulating requirements. The product owner requested that selected behavior be retained or accounted for, including licence-key compatibility and account linking through Telegram, Discord, or both. This context explains the source of the requirements; the qualification project focuses on the separate micro-SaaS platform built for the existing product.

The current production system contains four API services: user, license, usage, and distribution. User, license, and usage have separate PostgreSQL databases; distribution handles temporary files without its own database. RabbitMQ carries asynchronous commands and request/reply messages for file delivery. The web client uses React and TypeScript; nginx routes API traffic; Docker Compose runs the containers. These are current implementation facts; proposed future changes are listed separately.

The primary Appendix 1 direction for this project is distributed systems and Cloud-Native microservice architectures: the platform has services with separate responsibilities and data stores, and asynchronous communication through RabbitMQ. AI & Data Engineering, Application Security & DevSecOps, and DevOps & Quality Engineering are related to the platform's analytics, protection, and verification, but do not define the main architectural problem. Micro-SaaS is the focused service model; microservices describe how the platform is built.

## 4. Platform functions

1. **Product presentation and distribution:** product page, description, screenshots, free PC/mobile files, and guides.
2. **Plans and external purchase channels:** MTGVIP terms, links to purchase channels, source records, and activation of the acquired key.
3. **Accounts and dashboard:** Telegram and Discord sign-in, linked accounts, subscription and key status, and access history.
4. **Licensing and devices:** key generation and activation, expiry checks, HWID records, and tariff-based device quotas.
5. **Protected paid-file delivery:** access validation and asynchronous generation of a temporary VIP file with an expiry date.
6. **Telemetry and open usage analytics:** launch-event intake and aggregate reporting by time, server, faction, device, and version. Raw HWIDs are not public.
7. **Sales analytics and integrations:** aggregate sales, plan, and channel metrics; administrative API and Telegram/Discord bots.

## 5. Technology stack

| Category | Implementation |
|---|---|
| Languages | Python 3.12, TypeScript, SQL, Bash |
| Architectural patterns and interfaces | Microservices; REST APIs; asynchronous messaging; request/reply messaging |
| Backend frameworks and libraries | FastAPI, SQLAlchemy 2, Pydantic, HTTPX, aio-pika |
| Web frameworks and libraries | React 19, Vite, TanStack Query, Zustand, React Router, Tailwind CSS, Recharts, i18next |
| Integrations | python-telegram-bot, discord.py |
| Databases | PostgreSQL 16: separate stores for user, license, and usage services |
| Broker | RabbitMQ 3.13, AMQP |
| DevOps and deployment tools | Docker, Docker Compose, nginx |

## 6. Scope and engineering work

The project covers analysis of requirements based on the previous system and the product owner's request, and examination of the resulting micro-SaaS platform. Its engineering focus is the rationale for the microservice decomposition and component interactions, licensing and subscription mechanisms, account and device linking, file delivery, integrations, and analytics.

Proposed, measurable engineering tasks for supervisor approval include automated tests for key, subscription, and HWID flows; reliability testing of asynchronous file delivery under duplicate requests, delays, and failures; controlled load measurement of analytics APIs; and review of protections for sensitive telemetry and secrets. These are thesis proposals, not claims that the checks have already been implemented. The exact scope and evaluation criteria require supervisor approval.
