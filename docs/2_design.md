# РОЗДІЛ 2. ПРОЄКТУВАННЯ АРХІТЕКТУРИ СИСТЕМИ

Архітектура деталізує [SRS](srs.md) і зберігає межу між готовим цифровим продуктом та платформою його супроводу. У ЛР №5 побудовано три рівні C4 і зафіксовано [ADR-001](adr/0001-choice-of-technology.md). Схеми описують структурні рішення; вони не є доказом виконання цільових метрик або всіх принципів Clean Architecture.

Використано C4-семантику в Mermaid flowchart: Person — людина, System_Ext — зовнішня система, Software System — межа платформи, Container — застосунок/сховище, Component — внутрішня одиниця сервісу. Така нотація зберігає три рівні C4 і підтримує Mermaid Preview. Стрілки означають ініціатора виклику або залежність, а не повний порядок виконання; зворотні відповіді HTTP не дублюються. C4-контейнер не обов’язково дорівнює Docker-контейнеру: SPA та файловий каталог також можуть бути структурними елементами.

## 2.1. Контекстне та контейнерне моделювання системи (C4 Model)

<a id="diagram-level1"></a>
### C4 Level 1 — System Context

![C4 Context — статичне прев’ю](diagrams/c4-context.svg)

```mermaid
flowchart LR
  Guest["Person: Відвідувач"]
  User["Person: Користувач"]
  Admin["Person: Адміністратор"]
  Platform["Software System: micro-SaaS-платформа<br/>Представлення, доступ, файли та аналітика"]
  Product["System_Ext: Готовий SA-MP-продукт"]
  Telegram["System_Ext: Telegram<br/>Вхід, Stars, VIP-чат"]
  Discord["System_Ext: Discord<br/>Вхід, ролі, повідомлення"]
  Market["System_Ext: Зовнішній канал придбання"]
  Guest -->|"HTTPS: публічний вебінтерфейс"| Platform
  User -->|"HTTPS: браузер або Mini App"| Platform
  Admin -->|"HTTPS: адміністративний API"| Platform
  Product -->|"REST / HTTPS: check і launch"| Platform
  Platform -->|"HTTPS: Login / Bot API"| Telegram
  Platform -->|"OAuth2 / HTTPS: API"| Discord
  Platform -->|"HTTPS: зовнішній перехід"| Market
  Admin -->|"HTTPS: перевірка придбання"| Market
  classDef system fill:#e8f1ff,stroke:#2563eb,color:#111827
  class Platform system
```

[Окремий код Mermaid](diagrams/c4-context.mmd) · [Окрема схема в GitHub](diagrams/c4-context.md).

- Відвідувач працює з публічними сторінками й Free-файлами; користувач — з кабінетом, адміністратор — з операціями керування. Канал доступу — HTTPS.
- Зовнішній клієнт продукту викликає check і launch через REST/HTTPS; розроблення його ігрової логіки не належить платформі.
- Telegram забезпечує відповідні механізми входу, Stars і VIP-чат; Discord — OAuth2 та API ролей/повідомлень. Платформа звертається до API через HTTPS.
- Зовнішній канал придбання не є платіжним сервісом усередині платформи. HTTPS-перехід і ручна перевірка адміністратором не означають автоматичний webhook або інтегрований еквайринг.

Контекст покриває [FUN-01/02](srs.md#srs-fun-01), [вхід та ідентичності](srs.md#srs-fun-03), [клієнтську перевірку](srs.md#srs-fun-09) і [придбання](srs.md#srs-fun-16).

<a id="diagram-level2"></a>
### C4 Level 2 — Container

![C4 Container — статичне прев’ю](diagrams/c4-container.svg)

```mermaid
flowchart TB
  Browser["Person: Відвідувач / користувач / адміністратор<br/>Браузер або Telegram Mini App"]
  Product["System_Ext: Готовий SA-MP-продукт"]
  TG["System_Ext: Telegram API"]
  DC["System_Ext: Discord API"]
  Market["System_Ext: Зовнішнє придбання"]
  Gateway["Infrastructure: Host nginx<br/>HTTPS, reverse proxy, rate limits"]
  subgraph Platform["Software System: micro-SaaS-платформа"]
    Web["Container: Web<br/>React SPA / nginx"]
    UserAPI["Container: User Service<br/>FastAPI: ідентичності / сесії"]
    License["Container: License Service<br/>FastAPI: доступ / пристрої / продажі"]
    Usage["Container: Usage Service<br/>FastAPI: події / агрегати"]
    Dist["Container: Distribution Service<br/>FastAPI: VIP-збірки / завантаження"]
    UDB[("Data store: PostgreSQL<br/>Користувачі / сесії")]
    LDB[("Data store: PostgreSQL<br/>Ліцензії / пристрої / придбання")]
    SDB[("Data store: PostgreSQL<br/>Запуски продукту")]
    Redis[("Container: Redis<br/>Кеш агрегатів / refresh locks")]
    MQ["Container: RabbitMQ<br/>RPC збірки / команди ботам"]
    TBot["Container: Telegram bot<br/>Stars / VIP-чат"]
    DBot["Container: Discord bot<br/>VIP-ролі / повідомлення"]
    Files[("Data store: Файли<br/>Приватні шаблони / тимчасові збірки")]
    Free[("Data store: Free EXE<br/>Read-only базовий інсталятор")]
  end
  Browser -->|"HTTPS: HTML / JS / REST"| Gateway
  Product -->|"REST / HTTPS: check / launch"| Gateway
  Gateway -->|"HTTP: SPA / статичні файли"| Web
  Gateway -->|"HTTP: auth / profile"| UserAPI
  Gateway -->|"HTTP: license / sales"| License
  Gateway -->|"HTTP: usage"| Usage
  Gateway -->|"HTTP: VIP download"| Dist
  UserAPI -->|"SQL / PostgreSQL TCP"| UDB
  License -->|"SQL / PostgreSQL TCP"| LDB
  Usage -->|"SQL / PostgreSQL TCP"| SDB
  License -->|"HTTP / JSON: x-internal-token"| UserAPI
  License -->|"RESP / TCP: кеш продажів"| Redis
  Usage -->|"RESP / TCP: кеш використання"| Redis
  License -->|"AMQP / TCP: build request, reply queue"| MQ
  MQ -->|"AMQP / TCP: request / reply"| Dist
  License -->|"AMQP / TCP: bot commands"| MQ
  MQ -->|"AMQP / TCP: telegram.#"| TBot
  MQ -->|"AMQP / TCP: discord.#"| DBot
  TBot -->|"HTTP / JSON: x-bot-token"| License
  DBot -->|"HTTP / JSON: x-bot-token"| License
  UserAPI -->|"HTTPS: Telegram login"| TG
  UserAPI -->|"OAuth2 / HTTPS"| DC
  TBot -->|"HTTPS: Bot API / Stars"| TG
  DBot -->|"HTTPS: Discord API"| DC
  Web -->|"POSIX read: static Free EXE"| Free
  Dist -->|"POSIX read: stamping base"| Free
  Dist -->|"POSIX read / write / unlink"| Files
  Browser -->|"HTTPS: зовнішня оплата"| Market
```

[Окремий код Mermaid](diagrams/c4-container.mmd) · [Окрема схема в GitHub](diagrams/c4-container.md).

- **Gateway / host nginx** завершує TLS, маршрутизує HTTP у приватний контур і застосовує per-IP limits. Він є інфраструктурою доступу поза Docker Compose. [SEC-01](srs.md#srs-nfr-sec-01), [SEC-04](srs.md#srs-nfr-sec-04).
- **Web** — React SPA і nginx для статичного вмісту. Браузер виконує JavaScript та викликає API через gateway; стрілка gateway → Web означає доставку застосунку. Mini App відкриває той самий SPA. [FUN-01](srs.md#srs-fun-01), [FUN-02](srs.md#srs-fun-02).
- **User Service** зберігає користувачів, соціальні ідентичності й сесії у власній PostgreSQL; Telegram/Discord викликаються через HTTPS. [FUN-03](srs.md#srs-fun-03), [FUN-04](srs.md#srs-fun-04), [FUN-05](srs.md#srs-fun-05), [FUN-20](srs.md#srs-fun-20), [FUN-21](srs.md#srs-fun-21), [FUN-22](srs.md#srs-fun-22).
- **License Service** застосовує правила ключів, підписок, пристроїв і придбання; власна PostgreSQL містить відповідні записи. Профілі та social IDs отримуються від User Service через внутрішній HTTP з окремим токеном. [FUN-06](srs.md#srs-fun-06), [FUN-07](srs.md#srs-fun-07), [FUN-08](srs.md#srs-fun-08), [FUN-09](srs.md#srs-fun-09), [FUN-10](srs.md#srs-fun-10), [FUN-11](srs.md#srs-fun-11), [FUN-16](srs.md#srs-fun-16), [FUN-17](srs.md#srs-fun-17), [FUN-18](srs.md#srs-fun-18), [FUN-24](srs.md#srs-fun-24).
- **Usage Service** приймає події запуску й агрегує власну PostgreSQL. Публічні дані проходять через Redis-кеш; License Service використовує той самий Redis-контейнер з окремими ключами. RESP/TCP — протокол клієнта Redis. [FUN-14](srs.md#srs-fun-14), [FUN-15](srs.md#srs-fun-15), [CACHE-01](srs.md#srs-nfr-cache-01), [PRIV-01](srs.md#srs-nfr-priv-01).
- **Distribution Service** читає приватні шаблони й базовий EXE, записує тимчасові Lua/EXE та видає файл через HTTP API. Власна реляційна БД йому не потрібна. POSIX-позначення означає локальний файловий доступ, не мережевий протокол. [FUN-12](srs.md#srs-fun-12), [FUN-13](srs.md#srs-fun-13), [FILE-01](srs.md#srs-nfr-file-01).
- **RabbitMQ** переносить команди збірки та reply з URL, а також topic-команди ботам. HTTP-запит кабінету очікує RPC; брокер не робить його миттєвим. [REL-01](srs.md#srs-nfr-rel-01), [ADR-001](adr/0001-choice-of-technology.md).
- **Telegram/Discord-боти** споживають AMQP-команди й звертаються до License Service через HTTP з x-bot-token. Зовнішні API працюють через HTTPS; Stars належить Telegram-боту. [FUN-16](srs.md#srs-fun-16), [FUN-19](srs.md#srs-fun-19).

Три PostgreSQL не утворюють спільної ACID-транзакції; Redis не є первинним сховищем. Схема одного хоста не гарантує HA. Backup/restore й p95 перевіряються за [VC-08/11](srs.md#4-verification-criteria--критерії-верифікації), а не доводяться наявністю вузлів на рисунку.

## 2.2. Компонентне моделювання критичного сервісу системи

<a id="diagram-level3"></a>
### C4 Level 3 — License Service

Обрано License Service: у ньому зосереджено рішення про доступ, активацію, пристрої, придбання та видачу файлів. Нижче наведено **цільову проєктну модель**, уточнену за результатами аудиту ЛР №5. Вона визначає контракти для подальшої реалізації; наявність усіх портів і DI-provider у програмному коді не заявляється.

[Вихідний ескіз наявних залежностей](diagrams/c4-component-reference.mmd) зберігає concrete constructors, прямий download → messaging і worker orchestration для порівняння з результатом аудиту.

![C4 Component — цільова модель](diagrams/c4-component.svg)

```mermaid
flowchart TB
  Caller["External containers: Gateway / bots"]
  PG[("License PostgreSQL")]
  UserAPI["User Service"]
  MQ["RabbitMQ"]
  Redis[("Redis")]
  subgraph License["License Service — цільова компонентна модель"]
    subgraph Inbound["Inbound adapters"]
      API["REST controllers"]
      DTO["Pydantic DTO validation"]
      Guard["JWT / ADMIN / bot-token guards"]
      Worker["Expiry scheduler adapter"]
    end
    subgraph Core["Application / domain — без FastAPI та SQLAlchemy"]
      UseCases["License use cases<br/>activate / check / reset / expire"]
      Download["Download use case<br/>authorize + request build"]
      Stats["Sales statistics use case"]
      RP["Port: LicenseStore / PurchaseStore"]
      UP["Port: UserDirectory"]
      MP["Port: BuildGateway / BotPublisher"]
      CP["Port: StatsCache"]
    end
    subgraph Outbound["Outbound adapters"]
      Repo["SQLAlchemy repositories / transaction scope"]
      HTTP["httpx UserServiceClient"]
      AMQP["aio-pika RPC / publisher"]
      Cache["Redis cache adapter"]
    end
    Wiring["Composition root<br/>Depends providers / constructor injection"]
    Errors["Error boundary / sanitized logging"]
  end
  Caller -->|"HTTP / JSON"| API
  API -->|"FastAPI validation"| DTO
  API -->|"Depends: route-specific policy"| Guard
  API -->|"injected interface call"| UseCases
  API -->|"injected interface call"| Download
  API -->|"injected interface call"| Stats
  Worker -->|"injected expire use case"| UseCases
  UseCases -->|"interface call"| RP
  UseCases -->|"interface call"| UP
  UseCases -->|"interface call: expiry commands"| MP
  Download -->|"interface call: authorize"| RP
  Download -->|"interface call: build / deadline"| MP
  Stats -->|"interface call: aggregates loader"| RP
  Stats -->|"interface call: cache policy"| CP
  Repo -.->|"implements interface"| RP
  HTTP -.->|"implements interface"| UP
  AMQP -.->|"implements interface"| MP
  Cache -.->|"implements interface"| CP
  Wiring -.->|"binds ports to adapters"| Repo
  Wiring -.->|"binds ports to adapters"| HTTP
  Wiring -.->|"binds ports to adapters"| AMQP
  Wiring -.->|"binds ports to adapters"| Cache
  Wiring -.->|"constructs use cases"| UseCases
  Wiring -.->|"constructs use cases"| Download
  Wiring -.->|"constructs use cases"| Stats
  API -.->|"exception mapping / log event"| Errors
  Worker -.->|"exception mapping / log event"| Errors
  DTO -.->|"422 validation error"| Errors
  Repo -->|"SQL / PostgreSQL TCP"| PG
  HTTP -->|"HTTP / JSON: internal token"| UserAPI
  AMQP -->|"AMQP / TCP"| MQ
  Cache -->|"RESP / TCP"| Redis
```

[Окремий код Mermaid](diagrams/c4-component.mmd) · [Перегляд окремої схеми в GitHub](diagrams/c4-component.md).

### Компоненти, контракти та напрями залежностей

- **REST controllers** приймають HTTP і викликають application use cases через ін’єктовані залежності. Вони не виконують SQL і не звертаються до aio-pika напряму. Валідація **Pydantic DTO** та **JWT/ADMIN/bot-token guards** залишається на вході; публічні маршрути не отримують зайвого JWT-обмеження. [SEC-02](srs.md#srs-nfr-sec-02), [SEC-03](srs.md#srs-nfr-sec-03).
- **License use cases** визначають активацію, чинність, квоти, скидання й завершення ліцензії. Власник ресурсу перевіряється в use case; підпис JWT не підміняє ownership. [FUN-07](srs.md#srs-fun-07), [FUN-09](srs.md#srs-fun-09), [FUN-10](srs.md#srs-fun-10), [FUN-11](srs.md#srs-fun-11).
- **Download use case** об’єднує перевірку активного доступу та запит збірки через BuildGateway; загальний deadline керується в цьому application-сценарії. [FUN-12](srs.md#srs-fun-12), [REL-05](srs.md#srs-nfr-rel-05).
- **Sales statistics use case** звертається до портів сховища й кешу; loader-логіка не отримує мережевих деталей Redis. [FUN-17](srs.md#srs-fun-17), [CACHE-02](srs.md#srs-nfr-cache-02).
- **Expiry scheduler adapter** лише запускає відповідний use case. Зміна стану, отримання social IDs і публікація бот-команди координуються application layer через порти. [FUN-19](srs.md#srs-fun-19), [FUN-23](srs.md#srs-fun-23).
- **LicenseStore/PurchaseStore, UserDirectory, BuildGateway/BotPublisher, StatsCache** є внутрішніми контрактами. Use cases залежать від цих контрактів; адаптери реалізують їх. Пунктир implements означає залежність типів/реалізації, а не доставку повідомлення.
- **SQLAlchemy repositories / transaction scope** реалізують доступ до PostgreSQL; **httpx client** — HTTP до User Service; **aio-pika adapter** — AMQP RPC/publish; **Redis adapter** — RESP і cache locks. Контракти повинні повертати application DTO/значення й доменні помилки, не ORM-моделі або FastAPI Response.
- **Composition root** зв’язує порти з concrete adapters і створює use cases через constructor injection. FastAPI Depends використовується на межі застосунку; core не залежить від FastAPI, AsyncSession чи aio-pika. Різні порти для build і bot commands не вимагають спільного широкого інтерфейсу.
- **Error boundary / sanitized logging** перетворює помилки у визначений HTTP-контракт і записує технічну подію без секретів. Guards, валідація й error mapping є cross-cutting механізмами, а не бізнес-правилами. [OBS-01](srs.md#srs-nfr-obs-01), [PRIV-02](srs.md#srs-nfr-priv-02).

### Усунені недоліки на рівні проєктування

Після аудиту concrete construction перенесено в composition root, download orchestration — до use case, а ORM/HTTP/AMQP/Redis — за outbound ports. Це виправлення Mermaid-моделі та контрактів, не звіт про рефакторинг реалізації. Перевірка майбутньої реалізації має включати unit tests з підміною портів, перевірку імпортів core без framework dependencies та інтеграційні сценарії [VC-03–07](srs.md#4-verification-criteria--критерії-верифікації).

Transactional outbox, кластерний broker і distributed tracing не додаються до складу системи. Ризик неатомарності БД/publish залишається явним компромісом [ADR-001](adr/0001-choice-of-technology.md); DI не усуває цей ризик автоматично.

## 2.3. Журнал архітектурних рішень (Architecture Decision Records)

[ADR-001: RabbitMQ для команд ботам і RPC підготовки VIP-файлів](adr/0001-choice-of-technology.md) містить Context, Alternatives, Decision, Consequences і статус Approved у межах навчального проєктування. Розглянуто також прямий REST і Apache Kafka; наведено ризики таймаутів, повторів, неатомарності БД/брокера й експлуатаційних витрат.

## Трасування та перевірка

- Вісім груп FR ЛР №4 деталізовано в [24 вимоги SRS-FUN](srs.md#2-functional-requirements--функціональні-вимоги); кожна група віднесена до вузла Level 2, а критичні сценарії License — до Level 3.
- Чисельні обмеження збережено в [SRS-NFR](srs.md#3-non-functional-requirements--нефункціональні-вимоги); окремі [Verification Criteria](srs.md#4-verification-criteria--критерії-верифікації) називають інструменти й негативні сценарії.
- Mermaid-код є первинним артефактом. SVG-прев’ю генеруються з нього; при зміні коду вони мають оновлюватися разом.
- Якщо вбудований GitHub Mermaid-renderer не завантажується, SVG вище забезпечують перегляд схем; той самий Mermaid-код можна відкрити в Mermaid Live. Статичне прев’ю не є доказом роботи інтерактивного renderer.
- Статус архітектурного етапу — In Progress: виконано документування, очікується рев’ю. Вимірювання NFR і виправлення обмежень коду не позначено завершеними.

## Джерела

- [C4 Model: рівні моделювання](https://c4model.com/diagrams).
- [C4 Container: застосунки та сховища](https://c4model.com/diagrams/container).
- [Mermaid: синтаксис](https://mermaid.js.org/intro/syntax-reference.html).
