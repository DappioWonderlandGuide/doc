---
tags: docs
---

# Workflow [WIP]

> TODO

### Anatomy of Gateway

> TODO: Update diagram

![](https://hackmd.io/_uploads/Skbcueoyi.jpg)

Let's take a closer look on Gateway module. There are 4 different components in order to make Gateway function:
  - **Builder (Gateway Client)**: Off-chain component that helps composing different DeFi actions
  - **Protocol(s)**: Off-chain component that packages the specific instruction set of each protocol
  - **Gateway Program**: On-chain program that receives and dispatches all the transactions, manages state, distributes fees
  - **Adapter Program(s)**: On-chain program that connects base program and Gateway program

While **Builder** and **Protocols** are packaged into a npm module, **Gateway (program)** and **Adapters** are Solana programs deployed by Dappio (sometimes third-party) to be invoked to interact with Base programs

### Workflow

![](https://hackmd.io/_uploads/rkZ6gRbSo.png)

We can separate these series of operations into 2 distinct sections: **off-chain part** and **on-chain part**.

#### Off-chain Part

- User assembles actions by invoking action setter and providing proper parameters
- Builder composes transactions

#### On-chain Part

- User sends transactions directly to Gateway program
- Gateway dispatches transactions to their corresponding adapters through CPI (cross-program invocation)
- Adapter invokes Base program through another CPI

### Deployed Adapters

| Program | ID | Type |
| - | - | - |
| Gateway | `GATEp6AEtXtwHABNWHKH9qeh3uJDZtZJ7YBNYzHsX3FS` | - |
| Adapter Raydium | `ADPT1q4xG8F9m64cQyjqGe11cCXQq6vL4beY5hJavhQ5` | Pool / Farm |
| Adapter Orca | `ADPTTyNqameXftbqsxwXhbs7v7XP8E82YMaUStPgjmU5` | Pool / Farm |
| Adapter Saber | `ADPT4GbWTs9DXxo91YGBjNntYwLpXxn4gEbxfnUPfQoB` | Pool / Farm |
| Adapter Lifinity | `ADPTF4WmNPebELw6UvnSVBdL7BAqs5ceg9tyrHsQfrJK` | Pool |
| Adapter Solend | `ADPTCXAFfJFVqcw73B4PWRZQjMNo7Q3Yj4g7p4zTiZnQ` | MoneyMarket |
| Adapter Francium | `ADPTax5HwQ2ZWVLmceCek8UrqMhwCy5q3SHwi8W71Kv2` | MoneyMarket |
| Adapter Larix | `ADPTLQQ1Bwgybb2qge7QKSW7woDrhEjcLWG642qP2X4` | MoneyMarket / Farm |
| Adapter Tulip | `ADPT9nhC1asRcEB13FKymLTatqWGCuZHDznGgnakWKxW` | MoneyMarket / Vault |
| Adapter Friktion | `ADPTzbsaBdXA3FqXoPHjaTjPfh9kadxxFKxonZihP1Ji` | Vault |
| Adapter Katana | `ADPTwDKJTizC3V8gZXDxt5uLjJv4pBnh1nTTf9dZJnS2` | Vault |
| Adapter NFTFinance | `ADPTyBr92sBCE1hdYBRvXbMpF4hKs17xyDjFPxopcsrh` | NFTFinance |
