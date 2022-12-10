---
tags: docs
---

# Technical verview [WIP]

**We want developers to use DeFi elements as Lego blocks.** Universal Rabbit Hole enables developers to utilize various DeFi elements, such as Pool, Farm, Vault, to compose more complex operations without handling enormous amount of client SDK and inconsistent interfaces. 

| Without URH | With URH |
| - | - |
| Protocols are **NOT** composable. Developers have to deal with new SDK when trying to integrate new protocol | **Protocols are composable**. Developers only have to learn the knowledge of Gateway client to start developing |
| ![](https://hackmd.io/_uploads/HykS0teHi.png) | ![](https://hackmd.io/_uploads/SJzgZC-Hj.png) |

> TODO: Redraw diagram

> TODO: Add read part

> TODO: Redraw diagram

> TODO: Add Navigator in diagram

### You Just Need One Universal Interface. No More SDKs.

- Universal Rabbit Hole consists of 4 modules.
  - 2 modules are exposed, 2 modules are invisible to developers
- Navigator and Gateway are the developer interfaces

| Module | Functionality | 

> TODO

![](https://hackmd.io/_uploads/rJbWYMd-o.jpg)

There are two modules in Universal Rabbit Hole: **Navigator** and **Gateway**:

#### Navigator (Reader)

Navigator is a Typescript client for instantiating various of kinds of DeFi protocols. You can use it as a standalone dependency in your own project or together with [Dappio Gateway](https://guide.dappio.xyz/the-universal-rabbit-hole).

#### Gateway (Writer)

Gateway is a **CaaS (Compasaility-as-a-Service)** that standardizes inter-protocol interaction on Solana to unlock the potential of composability. It has an universal interface for various Solana DeFi elements (Pool / Farm / MoneyMarket / Vault / Leveraged Farm / ...) and serves as a **common knowledge base** that helps Solana community learn and improve.
