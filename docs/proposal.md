# Thesis Proposal

**Кваліфікаційний проєкт бакалавра** · Спеціальність F2 «Інженерія програмного забезпечення» (попередній код 121)

**Автор:** Маргер Богдан Євгенович
**Науковий керівник:** Дмитращук Кристина Миколаївна
**Статус теми:** попередня; очікує змістовного погодження керівником.

## 1. Попередня назва

**Українською:** Розроблення micro-SaaS-платформи для розповсюдження, ліцензування та аналітики цифрового продукту на основі мікросервісної архітектури.

**English:** Development of a Micro-SaaS Platform for Digital Product Distribution, Licensing, and Analytics Using a Microservices Architecture.

## 2. Актуальність і проблема

Предметна область проєкту — надання вузькоспеціалізованого micro-SaaS-сервісу для управління цифровим продуктом: його розповсюдженням, підписками, ліцензіями, файлами доступу та аналітикою продажів і використання. У проєктному сценарії замовником є власник уже готового цифрового програмного продукту. Для конкретизації вимог розглядається завантажуваний помічник для гравців San Andreas Multiplayer (SA-MP), зокрема спільнот Arizona RP і Rodina RP. Помічник існував до створення нової платформи, має безплатну версію та платні можливості за моделлю freemium. Його розроблення не є метою кваліфікаційного проєкту.

До появи нової платформи замовник продавав доступ вручну через чати, а потім використовував монолітну систему обліку продажів і доступу. Частковий аналіз її функцій та обмежень, разом із запитом замовника, дає вимоги до нової інфраструктури: облік покупців, сум і каналів продажу; створення й перевірка ліцензійних ключів із клієнта цифрового продукту; керування строком підписки; підтримка різних каналів отримання ключа; перевірка права на платні файли; облік продажів і базова публічна статистика. Код попередньої системи слугує джерелом для аналізу та вимог, але новий backend не є простим перенесенням моноліту.

На Етапі 3 розпочалася розробка нової системи: у межах університетських робіт із мікросервісної архітектури створено MVP backend для цієї предметної області. Подальші лабораторні й проєктні роботи розвинули його: на Етапі 4 систему адаптовано для production, доповнено вебсайтом, кабінетом, керуванням користувачами, ліцензуванням, розповсюдженням файлів і телеметрією; на Етапі 5 вебклієнт, backend і ботів об'єднано в монорепозиторій. Отже, предмет диплома — саме еволюція нової micro-SaaS-платформи від MVP мікросервісного backend до production-системи. Ручний процес і моноліт є вихідним станом та джерелом вимог; готовий помічник — продуктом, для якого будується платформа.

Інженерна проблема полягає в перенесенні потрібних бізнес-сценаріїв з обмеженої системи продажів у розподілену платформу та в узгодженні між її сервісами облікових записів, підписок, ліцензій, перевірок пристроїв, видачі файлів, телеметрії й зовнішніх каналів придбання. На відміну від початкового моноліту, нова платформа також надає особистий кабінет, розповсюджує безплатні версії, видає обмежені в часі VIP-файли та формує відкриті агрегати використання й продажів. Сайт спрямовує користувача до зовнішніх каналів оплати та обліковує дані операцій, але сам не здійснює банківський еквайринг.

## 3. Ідея та архітектура

Ідея роботи — micro-SaaS-платформа для власника цифрового продукту, яка об'єднує його вебпредставлення, розповсюдження файлів, керування платним доступом і пристроями, а також аналітику. Термін **micro-SaaS** тут описує модель вузькоспеціалізованого програмного сервісу для підтримки продукту; **мікросервісна архітектура** описує побудову самої платформи. Це різні характеристики однієї системи.

Продукт-кейс — готовий SA-MP помічник із безплатною версією та підпискою MTGVIP. Сайт представляє його, надає файли й інструкції для ПК та телефона, показує тарифи й переходи до зовнішніх способів придбання, а також публічну агреговану статистику. Особистий кабінет дає змогу входити через Telegram або Discord, активувати ключ, переглядати підписку, отримувати VIP-файл і керувати прив'язаними пристроями. Telegram і Discord боти виступають інтеграційними адаптерами. Головна сторінка автора є супровідною частиною сайту, а не окремим об'єктом розроблення.

Еволюція системи у проєктному сценарії:

- **Етапи 0–2 — вихідний стан і вимоги.** Ручне надання доступу через чати змінюється монолітом для обліку покупців, перевірки ключів із клієнта й керування підписками. Ця система працювала в production близько року. Її функції та обмеження використовуються для формування вимог.
- **Етап 3 — MVP нового backend.** Університетський репозиторій мікросервісів започаткував нову реалізацію тієї самої предметної області. Навчальні роботи пройшли від модульного моноліту до декомпозиції на сервіси користувачів/ліцензування, генерації файлів та використання. Це старт нової платформи, а не окрема випадкова тема.
- **Етап 4 — production-платформа.** MVP розвинуто й адаптовано для роботи з реальним продуктом: додано RabbitMQ, сервіси ідентичності та ліцензування, окремі бази PostgreSQL, вебклієнт і кабінет, публічну аналітику, інтеграції та розгортання через Docker Compose і nginx. Відбувся перехід чинного сервісу з SQLite-моноліту на нові сервіси й сховища.
- **Етап 5 — консолідація.** Backend, вебклієнт і боти зібрано в одному репозиторії; продовжено узгодження контрактів і функцій платформи.

Поточна production-система містить чотири API-сервіси: user, license, usage і distribution. Сервіси user, license та usage мають окремі PostgreSQL бази; distribution обробляє тимчасові файли без власної бази даних. RabbitMQ забезпечує асинхронні команди та взаємодію запит-відповідь під час видачі файлу. Вебклієнт побудований на React і TypeScript, API проходять через nginx, контейнери запускаються Docker Compose. Це наявна реалізація; запропоновані подальші зміни позначаються окремо.

Основний напрям Додатка 1 до ЛР — розподілені системи та Cloud-Native мікросервісні архітектури. Micro-SaaS є сервісною моделлю платформи, а мікросервіси — її архітектурним рішенням. У дипломі аналізується перехід від вимог, сформованих частковим аналізом моноліту, через MVP backend до production-платформи.

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
| Архітектура | Micro-SaaS-модель; доменна декомпозиція на мікросервіси; REST API; асинхронні повідомлення |
| Backend | FastAPI, SQLAlchemy 2, Pydantic, HTTPX, aio-pika |
| Web | React 19, Vite, TanStack Query, Zustand, React Router, Tailwind CSS, Recharts, i18next |
| Інтеграції | python-telegram-bot, discord.py |
| Бази даних | PostgreSQL 16: окремі сховища user, license та usage сервісів |
| Брокер | RabbitMQ 3.13, AMQP |
| Розгортання | Docker, Docker Compose, nginx |

## 6. Обсяг і напрями інженерної роботи

Проєкт охоплює формування вимог на основі часткового аналізу попередньої системи, розроблення MVP нового мікросервісного backend у межах Етапу 3 та подальшу еволюцію до production micro-SaaS-платформи на Етапах 4–5. Поточна система є результатом цієї поетапної розробки й водночас основою для аналізу в дипломі.

Для погодження з керівником запропоновано перевірювані подальші задачі: автоматизувати тести критичних сценаріїв ключів, підписок і HWID; оцінити надійність асинхронної видачі файлу при повторних запитах, затримках і відмовах; виміряти продуктивність API аналітики під контрольованим навантаженням; перевірити захист чутливих телеметричних даних і секретів. Це пропозиції для дипломної роботи, а не твердження про вже реалізовані перевірки. Конкретний обсяг і критерії оцінювання мають бути погоджені з керівником.

---

# Thesis Proposal (English)

**Bachelor’s qualification project** · F2 Software Engineering (previous code 121)

**Author:** Bohdan Marher
**Supervisor:** Krystyna Dmytrashchuk
**Topic status:** preliminary; pending substantive supervisor approval.

## 1. Preliminary title

**Development of a Micro-SaaS Platform for Digital Product Distribution, Licensing, and Analytics Using a Microservices Architecture.**

## 2. Relevance and problem statement

The subject area is the provision of a focused micro-SaaS service for managing a digital product: distribution, subscriptions, licences, access files, and sales and usage analytics. In the project scenario, the client is the owner of an existing digital software product. The case used to make the requirements concrete is a downloadable assistant for San Andreas Multiplayer (SA-MP) players, including the Arizona RP and Rodina RP communities. The assistant existed before the new platform and has a free edition and paid MTGVIP features. Developing the assistant itself is outside the scope of this qualification project.

Before the new platform, the client sold access manually through chat groups and later used a monolithic system for sales and access management. A partial analysis of that system's functions and limitations, together with the client's request, yields requirements for the new infrastructure: buyer, amount, and sales-channel records; licence-key creation and validation from the digital product's client; subscription expiry management; support for multiple key-purchase channels; paid-file entitlement checks; sales records and basic public statistics. The previous system is a source for analysis and requirements; the new backend is not a direct port of the monolith.

Stage 3 began development of the new system: university coursework in microservices produced an MVP backend for this subject area. Later coursework and project work evolved it. In Stage 4, the system was adapted for production and extended with a website, dashboard, user management, licensing, file distribution, and telemetry. In Stage 5, the web client, backend, and bots were consolidated into a monorepository. The qualification project therefore covers the evolution of this new micro-SaaS platform from a microservice-backend MVP to a production system. The manual process and monolith are the initial state and source of requirements; the existing assistant is the product served by the platform.

The engineering problem is migrating required business flows from a limited sales system into a distributed platform and coordinating accounts, subscriptions, licences, device checks, file delivery, telemetry, and external purchase channels. Beyond the original monolith, the platform provides a user dashboard, free-edition distribution, time-limited VIP files, and public aggregate usage and sales analytics. The website routes users to external payment channels and records transaction information; it does not process bank-card payments itself.

## 3. Concept and architecture

The project develops a micro-SaaS platform for a digital-product owner. It brings together web presentation, file distribution, paid-access and device management, and analytics. **Micro-SaaS** describes the focused service model around the product; **microservices** describe how the platform is architected. These are different properties of the same system.

The product case is an existing SA-MP assistant with a free edition and MTGVIP subscription. The website presents it, provides PC and mobile files and guides, lists plans and external purchase options, and publishes aggregate statistics. The user dashboard supports Telegram or Discord sign-in, key activation, subscription status, VIP-file downloads, and linked-device management. Telegram and Discord bots serve as integration adapters. The creator's home page supports the website but is not a separate development object.

The system evolved through these stages:

- **Stages 0–2 — starting point and requirements.** Access was first sold manually through chats, then managed in a monolith that recorded buyers, checked keys from the product client, and handled subscriptions. It ran in production for about a year. Its functions and limitations inform requirements.
- **Stage 3 — new backend MVP.** A university microservices repository began the new implementation for the same subject area. Coursework moved from a modular monolith to service decomposition for identity/licensing, file generation, and usage. This is the start of the current platform.
- **Stage 4 — production platform.** The MVP was adapted and extended for production: RabbitMQ, identity and licensing services, separate PostgreSQL databases, a web client and dashboard, public analytics, integrations, and Docker Compose/nginx deployment. The operating service moved from the SQLite monolith to new services and stores.
- **Stage 5 — consolidation.** Backend, web client, and bots were brought into one repository while platform contracts and functionality continued to evolve.

The current production system contains four API services: user, license, usage, and distribution. User, license, and usage have separate PostgreSQL databases; distribution handles temporary files without its own database. RabbitMQ carries asynchronous commands and request/reply messages for file delivery. The web client uses React and TypeScript; nginx routes API traffic; Docker Compose runs the containers. These are current implementation facts; proposed future changes are listed separately.

The primary Appendix 1 direction for this project is distributed systems and Cloud-Native microservice architectures. Micro-SaaS is the platform's service model; microservices are its architectural choice. The thesis analyses the progression from requirements derived through partial monolith analysis, through the backend MVP, to the production platform.

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
| Architecture | Micro-SaaS model; domain-oriented microservices; REST APIs; asynchronous messaging |
| Backend | FastAPI, SQLAlchemy 2, Pydantic, HTTPX, aio-pika |
| Web | React 19, Vite, TanStack Query, Zustand, React Router, Tailwind CSS, Recharts, i18next |
| Integrations | python-telegram-bot, discord.py |
| Databases | PostgreSQL 16: separate stores for user, license, and usage services |
| Broker | RabbitMQ 3.13, AMQP |
| Deployment | Docker, Docker Compose, nginx |

## 6. Scope and engineering work

The project covers requirements derived from a partial analysis of the previous system, development of a new microservice-backend MVP during Stage 3, and its subsequent evolution into a production micro-SaaS platform during Stages 4–5. The current system is the result of this staged development and the basis for the thesis analysis.

Proposed, measurable engineering tasks for supervisor approval include automated tests for key, subscription, and HWID flows; reliability testing of asynchronous file delivery under duplicate requests, delays, and failures; controlled load measurement of analytics APIs; and review of protections for sensitive telemetry and secrets. These are thesis proposals, not claims that the checks have already been implemented. The exact scope and evaluation criteria require supervisor approval.
