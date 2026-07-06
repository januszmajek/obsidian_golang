---
date: 2026-07-06
---
# WMS Business Process Flow

```mermaid
flowchart LR

    P[Products]
    IO[Inbound Operations]
    S[(Stock)]
    O[Orders]
    OI[Order Items]
    OO[Outbound Operations]

    P --> IO
    IO -->|Increase Quantity| S

    O --> OI
    P --> OI

    OI -->|Consume Stock| S

    O --> OO
    OO -->|Ship Order| S
```
