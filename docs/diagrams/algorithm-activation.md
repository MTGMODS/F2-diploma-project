# A1 — Активація ліцензії

[Інтерактивний перегляд у Mermaid Live](https://mermaid.live/view#pako:eJx9VW1L21AU_iuXfFJWa1u2D6s4iEmKWbO0NmnmSEsISVyD9oU02SxRUD84BoIwhh8GY2O_oMjcxE39Czd_Yb9k594kbeJ0-XbveXKe55zn3HtDxhraDlNlXnvmqIdUvjNA8CkLOv6Cb6J3eBod4Rt81V1ES0vPkKgv4w_4PDrD36vo-Uu1gLadSQFtDT3LWe7G_4oUqem6hqqrSDN3XNv0HQCzA5tXG90EplHYRqgVh9toFamttrAfRzZIZA9_js72kCZQxsuUUyv2fH9kjH3TD8YpoybQXAKI_gSwa3wRHUcn3cVcum94iq_2kKTrEtFVcwe25FrOYOysTerOJJUl0VT1UEJ_3n9FcluSECvzSCrGlKBUbqgGy6mixqoCn0iuZySvsXxe8-NSCYmyxkoib9SFV6lqwMWycykSmayuN3bsVChr-e4bJ5FbG3rtseOlglmahAsJPCcZdMbGJBq5HAHXkO-qfIpoWYIhiZwgK4IhbIqKqqR64Y-sXi5T8nqOPeFbz_E1SD1pE6EsYbMptgR-BZFtZ3fkes7YMH0Skodv2yrXzWWhNMlwNqgMPpSKduCZvjscGLY5odbM2fkcu6wLhGJC0wOomwPR5KreYTIgqgE9QjxkXrjDtNhhkgRxRxRBjZdqutSlYgAeGS61UEsXK_M5gu2422TPJP7CIbFzHSCRfGdifQk30MTWl3W9VSZhbtjvuz4bJwOx6YhwZQps1Mthq_zPcYPtTBtq-al4UiqtoOgQLoEpvkYWJUD4Fk7YAb4goOggOo0-RkfRCaBOEbkraOg3vkkHp5Ydmxlb4kxT19XddM6bgWf1THIik1lPK2jSFOusEgL47pyxSkZ_S1DyFVTg9I0Dy3LGs_ti9keiod0EDQ8YBoG5Y1zjRVMSVDK2sD9K1Nr3zW07lsxVwJzK3Jy0wpk1lcSaStiq3GNNJScUirsToDU32ZZ6j23kHzAONuEWPwarwJ5z_ANfIPwTYrf4MjoktuJp2hiSKGsW8GWXcDcWEXkXwN1L_Atf4SkqQrg2u4n-F24-ECaknQFTYPqO1zddm6mGjN9z-uRVsp0tM9jxmf0CYwb-UJkMLKbqe4FTYIIReVZ414SXqx9v7v8F8V980Q). [Розділ 2](../2_design.md).

```mermaid
graph TD
    S([Початок]) --> I[/Ввід: JWT, key, force/]
    I --> V[[V := ValidateJWTAndDTO]]
    V --> Q{V.ok = TRUE}
    Q -->|Ні| VE[/Вивід: V.http_status/]
    VE --> E([Кінець])
    Q -->|Так| L[[L := FindLicenseByKey]]
    L --> K{L ≠ NULL AND L.status = NOT_ACTIVATED}
    K -->|Ні| BAD[/Вивід: 400 INVALID_KEY/]
    BAD --> E
    K -->|Так| A[[Old := FindActiveLicenseForUser]]
    A --> C{Old ≠ NULL AND NOT force}
    C -->|Так| CON[/Вивід: 409 ACTIVE_LICENSE_EXISTS/]
    CON --> E
    C -->|Ні| H{Old ≠ NULL}
    H -->|Так| O[Old.status := EXPIRED; Old.expires_at := NowUTC]
    H -->|Ні| D
    O --> D{L.duration_days = NULL}
    D -->|Так| N[Expiry := NULL]
    D -->|Ні| T["Expiry := NowUTC + Days(L.duration_days)"]
    N --> SET
    T --> SET[L.user_id := V.user_id; L.status := ACTIVE; L.activated_at := NowUTC; L.expires_at := Expiry]
    SET --> C1[[R1 := CommitActivation]]
    C1 --> OK1{R1.ok = TRUE}
    OK1 -->|Ні| F[/Вивід: 500; стан commit перевіряється окремо/]
    F --> E
    OK1 -->|Так| P[[Tx := FindPurchaseByLicense]]
    P --> HAS{Tx ≠ NULL}
    HAS -->|Ні| RES[/Вивід: 200 success/]
    HAS -->|Так| UP[Tx.user_id := V.user_id; Tx.status := COMPLETED; Tx.purchased_at := NowUTC]
    UP --> C2[[R2 := CommitPurchase]]
    C2 --> OK2{R2.ok = TRUE}
    OK2 -->|Так| RES
    OK2 -->|Ні| PART[/Вивід: 500; активація вже записана/]
    PART --> E
    RES --> E
    L -. Помилка .-> F
    A -. Помилка .-> F
    P -. Помилка .-> PART
```
