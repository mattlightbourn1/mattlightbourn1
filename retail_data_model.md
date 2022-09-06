
```mermaid
erDiagram
    CUSTOMER ||--o{ SALES-ORDER : places
    SALES-ORDER ||--|{ LINE-ITEM : contains
    LINE-ITEM ||--|{ PRODUCT : contains
    INVENTORY ||--|{ PRODUCT : contains
    CUSTOMER ||--|{ CATALOGUE : uses
    CATALOGUE ||--|{ PRODUCT : contains
```
