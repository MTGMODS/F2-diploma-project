# ERD — 7 таблиць у 3 PostgreSQL-БД

[Інтерактивний перегляд у Mermaid Live](https://mermaid.live/view#pako:eJy1VtuO2jAQ_RXLT10Jbauq7QNviKXVqoheKG9I1jSeEIvYjmyHhbL8eyeQtNxyoZe8REqOj2fOnBl7yyMrkfc5ugcFCwd6bhg9s-noq8g9Os-2hy_Fo0xgSrLPH9mcTz59Y5PZeDznv_9_V4sCEjDFgkoQdrbHEo5BmtonlFfwUvnIOtkKX4GLEnAv3r66Y0ZFSwMaayIpoQxWEMCJ3KUdeF8Tr7NpHWchh_ABQu5Z-boODEojAXQWfrDIIQSUAkJDAMcLUvBBpJaEqV-zOyqShTwkIgEjbRyfVKtK692bO9ohWmKor1zAdWAZbFILskNWuM6UQ19GeAY-js5hTLhEePReWXOTm4r_e81VXUgnGdolGtLBJ6WJ_kEK48fhaDIdiVRFaCiFv4i-2XpsiZv6sPc9kjsIpKCQsPENhAWW0kLykNLqWm4VSsNaSFxRanU-LtO-9HwHG9_se4iCWt225LSE9X1yVkZRbvUnfqwoCPe-w6hiB4FbhpTKBEhJmTSJW4H3noIFmq4qxcrRRKFltw6h5iVn2gYHxhfCkkWzvAj1v_TLRQ16jSdGTPMsMNA2N3WtUOlKw0-TqEJjSGzLwLmtFSo52sWcTQcfyKaQmyi5Tb0qsBWd2FSCFlQnVyZPtXO3iIZKRZt1MKymC0Zn2-0z7-66X4P52d7f221jp_eJ8sg6L9lwMB0OHkYV9XXO5xaHX7DOJo9fZqNjet7jGp0GJXl_y0OCurhwSYwhTwPf9Tid4Ha6MRHvB5djj-eZpEFY3scOH3c_Ad25-_E). [Розділ 2](../2_design.md).

```mermaid
erDiagram
    USER_users {
        int id PK "NOT NULL"
        bigint telegram_id UK "NULL allowed"
        bigint discord_id UK "NULL allowed"
        varchar(50) nickname "NOT NULL"
        varchar avatar_url "NULL allowed"
        varchar(20) role "NOT NULL"
        user_status status "NOT NULL"
        timestamptz created_at "NULL allowed"
        timestamptz last_login_at "NULL allowed"
    }
    USER_oauth_handoffs {
        varchar(64) ticket PK "NOT NULL"
        text payload "NOT NULL"
        timestamptz expires_at "NOT NULL"
    }
    USER_refresh_sessions {
        int id PK "NOT NULL"
        int user_id "NOT NULL"
        varchar(64) token_hash UK "NOT NULL"
        timestamptz expires_at "NOT NULL"
    }
    LICENSE_licenses {
        int id PK "NOT NULL"
        int user_id "NULL allowed"
        varchar key UK "NOT NULL"
        int duration_days "NULL allowed"
        int reset_limit "NOT NULL"
        int max_devices "NOT NULL"
        license_status status "NULL allowed"
        timestamptz created_at "NULL allowed"
        timestamptz activated_at "NULL allowed"
        timestamptz expires_at "NULL allowed"
    }
    LICENSE_license_activations {
        int id PK "NOT NULL"
        int license_id FK "NULL allowed"
        varchar device "NOT NULL"
        varchar ip_address "NULL allowed"
        varchar user_agent "NULL allowed"
        timestamptz first_used_at "NULL allowed"
        timestamptz last_used_at "NULL allowed"
    }
    LICENSE_transaction_purchases {
        int id PK "NOT NULL"
        int user_id "NULL allowed"
        int license_id FK,UK "NULL allowed"
        float amount "NOT NULL"
        varchar payment_method "NOT NULL"
        varchar status "NULL allowed"
        timestamptz purchased_at "NULL allowed"
    }
    USAGE_launches {
        int id PK "NOT NULL"
        varchar version "NOT NULL"
        varchar device "NOT NULL"
        varchar hwid "NOT NULL"
        int server "NULL allowed"
        varchar mode "NULL allowed"
        timestamptz launched_at "NULL allowed"
    }
    LICENSE_licenses |o..o{ LICENSE_license_activations : "license_id / CASCADE"
    LICENSE_licenses |o..o| LICENSE_transaction_purchases : "license_id / UNIQUE / CASCADE"
```
