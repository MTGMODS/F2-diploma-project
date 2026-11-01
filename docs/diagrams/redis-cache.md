# Redis — кеш і блокування

[Інтерактивний перегляд у Mermaid Live](https://mermaid.live/view#pako:eJx1UrtuwzAM_BWBaxOjj81Dl7ZLWrRAsxoQFIm21chSIFFpUiP_Xvkx2EjMiTgSdzySLUinEHKQRoTwqkXlRVNYlqJH2IuQNb7ZIxp3QNYOpS7u3O4HJTElSExQG5sdelZ6DDWPlrQZipcp6TcqHXrmdzzPSAN5bSu2x_M1uNl-ffKjMBEnNW0JqyTokTCpOcsDSmdVWJL9cHK_rMpDLEt94iZ1XTe4X4uek9uj5TWenh5vDEJkbo4w91zAQwFsvX4es_ma894rC-i1MPpPdL6WWbIsJfdZ1uUzi3naSn8IJp3zStuRCFbQoG-EVpC3QDU23QcoLEU0BJcViEhue7YScvIRVxAP6co4fscAXv4BGJa4CQ). [Розділ 2](../2_design.md).

```mermaid
classDiagram
    class CacheEnvelope {
        +object data
        +number fresh_until
    }
    class RedisCacheKey {
        +string key
        +string JSON_value
        +integer retention_seconds
    }
    class RedisLockKey {
        +string key_suffix_lock
        +string owner_token_hex32
        +integer ttl_seconds
    }
    RedisCacheKey "1" --> "1" CacheEnvelope : JSON serialization
    RedisCacheKey "1" .. "0..1" RedisLockKey : refresh coordination
```
