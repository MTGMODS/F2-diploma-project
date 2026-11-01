# Специфікація моделей даних

Три окремі PostgreSQL-БД: `user_db`, `license_db`, `usage_db`. Префікси в ERD позначають власника таблиці; це не спільна схема й не дозвіл на міжсервісні JOIN. Типи, nullable, індекси та default звірено з SQLAlchemy-моделями. DBML є декларативною копією моделі, а не міграцією для трьох БД.

`PK` включає NOT NULL. `UK` означає UNIQUE або унікальний індекс. У PostgreSQL UNIQUE допускає кілька NULL. `ORM default/onupdate` виконує SQLAlchemy; це не серверне обмеження. Необмежений `varchar` у схемі не означає відсутності DTO-обмежень.

Назви `user_status`/`license_status` у DBML та ERD є псевдонімами SQLAlchemy Enum(UserStatus/LicenseStatus), а не твердженням про ім’я фізичного PostgreSQL-типу.

## user_db.users

| Поле | Тип PostgreSQL / ORM | Обмеження | Призначення |
|---|---|---|---|
| `id` | `int` | PK; автоінкремент; NOT NULL | Внутрішній ID користувача. |
| `telegram_id` | `bigint` | NULL; UNIQUE index | ID Telegram; не внутрішній ID. |
| `discord_id` | `bigint` | NULL; UNIQUE index | ID Discord; не внутрішній ID. |
| `nickname` | `varchar(50)` | NOT NULL; index | Відображуване ім’я. |
| `avatar_url` | `varchar` | NULL | Адреса аватара. |
| `role` | `varchar(20)` | NOT NULL; ORM default USER | Роль доступу; звичайний рядок. |
| `status` | `user_status` | NOT NULL; ORM default ACTIVE | Стан облікового запису. |
| `created_at` | `timestamptz` | NULL дозволений; server default now() | Час створення, UTC. |
| `last_login_at` | `timestamptz` | NULL; ORM onupdate now(); початкового default немає | Час оновлення під час входу. |

**Індекси:** `ix_users_id` (id); `ix_users_telegram_id` (telegram_id) UNIQUE; `ix_users_discord_id` (discord_id) UNIQUE; `ix_users_nickname` (nickname). Індекси PK/UNIQUE створюються PostgreSQL окремо від цього переліку.

## user_db.oauth_handoffs

| Поле | Тип PostgreSQL / ORM | Обмеження | Призначення |
|---|---|---|---|
| `ticket` | `varchar(64)` | PK; NOT NULL | Одноразовий квиток обміну OAuth-результатом. |
| `payload` | `text` | NOT NULL | Серіалізований JSON результату; фізично TEXT. |
| `expires_at` | `timestamptz` | NOT NULL | Граничний час використання квитка. |

**Індекси:** окремих іменованих індексів немає. Індекси PK/UNIQUE створюються PostgreSQL окремо від цього переліку.

## user_db.refresh_sessions

| Поле | Тип PostgreSQL / ORM | Обмеження | Призначення |
|---|---|---|---|
| `id` | `int` | PK; автоінкремент; NOT NULL | ID refresh-сесії. |
| `user_id` | `int` | NOT NULL; index; FK відсутній | Логічний власник: users.id. |
| `token_hash` | `varchar(64)` | NOT NULL; UNIQUE | Хеш токена; відкритий токен не зберігається. |
| `expires_at` | `timestamptz` | NOT NULL | Час завершення сесії. |

**Індекси:** `ix_refresh_sessions_user_id` (user_id). Індекси PK/UNIQUE створюються PostgreSQL окремо від цього переліку.

## license_db.licenses

| Поле | Тип PostgreSQL / ORM | Обмеження | Призначення |
|---|---|---|---|
| `id` | `int` | PK; автоінкремент; NOT NULL | ID ліцензії; поле id відповіді check. |
| `user_id` | `int` | NULL до активації; index; без FK | Власник; логічне посилання на User Service. |
| `key` | `varchar` | NOT NULL; UNIQUE index; VARCHAR без DB CHECK довжини | Ключ формату XXXX-XXXX-XXXX-XXXX, A–Z/0–9. |
| `duration_days` | `int` | NULL; позитивність перевіряє API генерації | Тривалість у добах; NULL — безстрокова ліцензія. |
| `reset_limit` | `int` | NOT NULL; ORM default 1 | Кількість доступних скидань пристроїв. |
| `max_devices` | `int` | NOT NULL; ORM default 2 | Квота пристроїв. |
| `status` | `license_status` | NULL дозволений; ORM default NOT_ACTIVATED | Стан ліцензії. |
| `created_at` | `timestamptz` | NULL дозволений; server default now() | Час створення. |
| `activated_at` | `timestamptz` | NULL до активації | Час активації. |
| `expires_at` | `timestamptz` | NULL | Час закінчення; NULL для безстрокової. |

**Індекси:** `ix_licenses_id` (id); `ix_licenses_user_id` (user_id); `ix_licenses_key` (key) UNIQUE. Індекси PK/UNIQUE створюються PostgreSQL окремо від цього переліку.

## license_db.license_activations

| Поле | Тип PostgreSQL / ORM | Обмеження | Призначення |
|---|---|---|---|
| `id` | `int` | PK; автоінкремент; NOT NULL | ID запису пристрою. |
| `license_id` | `int` | NULL; FK licenses.id; ON DELETE CASCADE | Ліцензія пристрою. |
| `device` | `varchar` | NOT NULL; API check: 2–100 символів | Ідентифікатор HWID; не тип PC/MOBILE. |
| `ip_address` | `varchar` | NULL; VARCHAR, не тип INET | IP останньої перевірки. |
| `user_agent` | `varchar` | NULL | User-Agent клієнта. |
| `first_used_at` | `timestamptz` | NULL дозволений; server default now() | Час першого звернення пристрою. |
| `last_used_at` | `timestamptz` | NULL дозволений; server default now(); ORM onupdate now() | Час останнього звернення. |

**Індекси:** `ix_license_activations_id` (id). Індекси PK/UNIQUE створюються PostgreSQL окремо від цього переліку.

## license_db.transaction_purchases

| Поле | Тип PostgreSQL / ORM | Обмеження | Призначення |
|---|---|---|---|
| `id` | `int` | PK; автоінкремент; NOT NULL | ID запису продажу. |
| `user_id` | `int` | NULL; index; без FK | Власник покупки; логічний users.id. |
| `license_id` | `int` | NULL; UNIQUE; FK licenses.id; ON DELETE CASCADE | Ліцензія; не більше однієї покупки для ліцензії. |
| `amount` | `float` | NOT NULL; SQLAlchemy Float; неточна двійкова арифметика | Сума зовнішнього продажу. |
| `payment_method` | `varchar` | NOT NULL; VARCHAR | Канал зовнішньої оплати; не еквайринг сайту. |
| `status` | `varchar` | NULL дозволений; ORM default COMPLETED; VARCHAR, не ENUM | Стан обліку покупки, зокрема PENDING/COMPLETED. |
| `purchased_at` | `timestamptz` | NULL дозволений; server default now() | Час обліку/завершення покупки. |

**Індекси:** `ix_transaction_purchases_id` (id); `ix_transaction_purchases_user_id` (user_id). Індекси PK/UNIQUE створюються PostgreSQL окремо від цього переліку.

## usage_db.launches

| Поле | Тип PostgreSQL / ORM | Обмеження | Призначення |
|---|---|---|---|
| `id` | `int` | PK; автоінкремент; NOT NULL | ID події запуску. |
| `version` | `varchar` | NOT NULL; обмеження формату на вході Usage API | Версія та редакція цифрового продукту. |
| `device` | `varchar` | NOT NULL; VARCHAR; перевірка API, не DB ENUM | Категорія пристрою PC або MOBILE. |
| `hwid` | `varchar` | NOT NULL; index; Usage API: 5–255 символів | HWID події, а не FK до пристрою ліцензії. |
| `server` | `int` | NULL; index | Числовий код ігрового сервера. |
| `mode` | `varchar` | NULL; index | Режим використання. |
| `launched_at` | `timestamptz` | NULL дозволений; server default now(); index | Час отримання події запуску. |

**Індекси:** `ix_launches_id` (id); `ix_launches_hwid` (hwid); `ix_launches_server` (server); `ix_launches_mode` (mode); `ix_launches_launched_at` (launched_at). Індекси PK/UNIQUE створюються PostgreSQL окремо від цього переліку.

## Зв’язки та словники

Є лише два фізичні FK, обидва всередині License DB. Для кожної активації або покупки допускається 0..1 ліцензія, оскільки FK nullable; у ліцензії 0..N записів пристроїв і 0..1 покупка. Власні PK дочірніх таблиць не включають FK, тому зв’язки неідентифікувальні (`..`). Пунктир не означає відсутність фізичного FK.

`refresh_sessions.user_id → users.id` — логічний зв’язок у User DB без FK. `licenses.user_id` і `transaction_purchases.user_id → users.id` — логічні міжсервісні посилання через HTTP User Service; PostgreSQL їх не контролює. `launches.hwid` не є FK до `license_activations.device`.

- `user_status`: ACTIVE, BANNED, DELETED.
- `license_status`: NOT_ACTIVATED, ACTIVE, EXPIRED, BANNED.
- `payment_method` — VARCHAR; API використовує Stars, FunPay, Crypto, PayPal, Card, Steam, Gift, Promo.
- Статус покупки — VARCHAR; COMPLETED не є значенням LicenseStatus.
- `Float` у SQLAlchemy компілюється у PostgreSQL як FLOAT/double precision; NUMERIC не заявляється як наявний тип.

## Нормалізація і консистентність

Атрибути таблиць залежать від їхніх ключів; масивів пристроїв у рядку ліцензії немає. Для наведених функціональних залежностей не виявлено порушення 3NF. `max_devices`, `reset_limit`, `duration_days` — знімок індивідуальних умов ліцензії, а не довідник тарифів. `oauth_handoffs.payload` є непрозорим короткоживучим транспортним текстом: він не використовується для реляційного пошуку. Повторення user_id у ліцензії та покупці пов’язує різні бізнес-факти; узгодженість забезпечує активаційний сценарій.

Відсутність міжсервісних FK є свідомою межею володіння даними, але локальний FK refresh_sessions.user_id можна розглянути після перевірки orphan-рядків. Видалення користувача не повинно неявно каскадувати історію продажів. Зберігання часу — timezone-aware UTC; представлення API/AMQP — ISO 8601, `Z`.

## План індексування та інженерний борг

| Запит / ризик | Наявна підтримка | Рішення та спосіб перевірки |
|---|---|---|
| Ключ ліцензії, соціальний ID, refresh-хеш | UNIQUE B-tree | Зберегти; перевірити індексний пошук на тестових даних |
| Активна ліцензія за користувачем | B-tree user_id; фільтр status | Перевірити `(user_id, status)` або частковий індекс ACTIVE через EXPLAIN (ANALYZE, BUFFERS); не створювати без вимірювань |
| Пристрої конкретної ліцензії | Окремого індексу license_id немає | Кандидат `(license_id, device)`; унікальність лише після усунення дублів і nullable-рядків |
| Конкурентна активація/квота | UNIQUE key не серіалізує зміну власника | Критерій приймання: транзакційна синхронізація та 20 конкурентних спроб без подвійної активації; схема не доводить виконання |
| Підрядковий ILIKE nickname | Звичайний B-tree nickname | Для `%term%` розглянути pg_trgm GIN після EXPLAIN; звичайний B-tree не гарантує прискорення |
| Продажі за часом, Usage-агрегати | Launches: hwid/server/mode/launched_at; purchases: user_id | Порівняти план групування, часові діапазони й обсяг записів; складені/BRIN індекси лише за профілем навантаження |
| Дублювання PK-індексів | Частина id має окремий index=True | Перевірити pg_indexes, залежності та статистику перед вилученням; нічого не видалено |
| Грошові суми Float | Неточне двійкове число | Рекомендовано NUMERIC(p,s) після вибору валюти/масштабу і міграційної перевірки; вихідний тип у ERD збережено |
| Nullable status/FK; відсутні CHECK | Значення частково обмежує API | Спочатку аудит даних; потім оцінити NOT NULL, CHECK квот/тривалості та history/audit. Це пропозиції, не чинні обмеження |

Повний перелік містить 7 таблиць, 47 полів, 16 іменованих вторинних індексів і 2 фізичні FK. EXPLAIN, міграції та навантажувальні вимірювання в межах ЛР №6 не виконувалися.
