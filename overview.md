---
tags: docs
---

# Technical verview [WIP]

**We want developers to use DeFi elements as Lego blocks.** Universal Rabbit Hole enables developers to utilize various DeFi elements, such as Pool, Farm, Vault, to compose more complex operations without handling enormous amount of client SDK and inconsistent interfaces. 

> Replace Different SDKs with One Universal Interface

| Without URH | With URH |
| - | - |
| Protocols are **NOT** composable. Developers have to deal with new SDK when trying to integrate new protocol | **Protocols are composable**. Developers only have to learn the knowledge of Gateway client to start developing |
| ![](https://hackmd.io/_uploads/HykS0teHi.png) | ![](https://hackmd.io/_uploads/SJzgZC-Hj.png) |

> TODO: Redraw diagram

![](https://hackmd.io/_uploads/SkfiOPQOo.png)

<!-- ![](https://hackmd.io/_uploads/ryy8kw7uj.png) -->

<!-- ![](https://hackmd.io/_uploads/rJbWYMd-o.jpg) -->

Universal Rabbit Hole is a **CaaS (Compasaility-as-a-Service)** that standardizes inter-protocol interaction on Solana to unlock the potential of composability. It has an universal interface for various Solana DeFi elements (Pool / Farm / MoneyMarket / Vault / Leveraged Farm / ...) and serves as a **common knowledge base** that helps Solana community learn and improve.

| Module | Type | Functionality |
| - | - | - |
| Navigator | Typescript Client | Navigator is a Typescript client for instantiating various of kinds of DeFi protocols. You can use it as a standalone dependency in your own project or together with Dappio Gateway. |
| Gateway | Typescript Client | Gateway client is an off-chain component that helps composing instructions for supported protocols |
| Gateway-program | On-chain program | On-chain program that receives and dispatches all the transactions, manages state, distributes fees |
| Adapter-programs | On-chain program | On-chain program that connects base program and Gateway program |

While **Navigator** and **Gateway Client** are packaged into a npm module, **Gateway Program** and **Adapter Programs** are Solana programs deployed by Dappio (sometimes third-party) to be invoked to interact with Base programs

- Universal Rabbit Hole consists of 4 modules.
  - 2 modules are exposed, 2 modules are invisible to developers
- Navigator and Gateway are the developer interfaces
- See quickstart example for more details

### Anatomy of the Gateway and Adapters

![](https://hackmd.io/_uploads/ryFDCDQui.png)

![](https://hackmd.io/_uploads/rkZ6gRbSo.png)

Let's take a closer look on the *write process* of Universal Rabbit Hole.

We can separate these series of operations into 2 distinct sections: **off-chain part** and **on-chain part**.

### Off-chain Part

- User assembles actions by invoking action setter and providing proper parameters
- Builder composes transactions

### On-chain Part

- User sends transactions directly to Gateway program
- Gateway dispatches transactions to their corresponding adapters through CPI (cross-program invocation)
- Adapter invokes Base program through another CPI
