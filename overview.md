---
tags: docs
---

# Overview [WIP]

> TODO

> See [this guide](https://guide.dappio.xyz/the-universal-rabbit-hole) and [this Medium Post](https://medium.com/dappio-wonderland/the-solution-to-composability-universal-rabbit-hole-28b817cc0fd4) for more details

### What Does URH Solve?

#### Without URH

> TODO: Redraw diagram

> TODO: Add read part

![](https://hackmd.io/_uploads/HykS0teHi.png)

- **Protocols are not composable**
- User has to deal with new SDK when trying to integrate new protocol

#### With URH

> TODO: Redraw diagram

> TODO: Add Navigator in diagram

![](https://hackmd.io/_uploads/SJzgZC-Hj.png)

- **Protocols are composable**
- User only has to deal with builder (Gateway client) and Gateway program

**We want developers to use DeFi elements as Lego blocks.** Universal Rabbit Hole enables developers to utilize various DeFi elements, such as Pool, Farm, Vault, to compose more complex operations without handling enormous amount of client SDK and inconsistent interfaces. 

### You Just Need One Universal Interface. No More SDKs.

> TODO

![](https://hackmd.io/_uploads/rJbWYMd-o.jpg)

There are two modules in Universal Rabbit Hole: **Navigator** and **Gateway**:

#### Navigator (Reader)

Navigator is a Typescript client for instantiating various of kinds of DeFi protocols. You can use it as a standalone dependency in your own project or together with [Dappio Gateway](https://guide.dappio.xyz/the-universal-rabbit-hole).

#### Gateway (Writer)

Gateway is a **CaaS (Compasaility-as-a-Service)** that standardizes inter-protocol interaction on Solana to unlock the potential of composability. It has an universal interface for various Solana DeFi elements (Pool / Farm / MoneyMarket / Vault / Leveraged Farm / ...) and serves as a **common knowledge base** that helps Solana community learn and improve.
