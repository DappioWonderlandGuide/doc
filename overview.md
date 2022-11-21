---
tags: docs
---

# Overview [WIP]

> TODO

**We want developers to use DeFi elements as Lego blocks.** Universal Rabbit Hole enables developers to utilize various DeFi elements, such as Pool, Farm, Vault, to compose more complex operations without handling enormous amount of client SDK and inconsistent interfaces. 

> See [this guide](https://guide.dappio.xyz/the-universal-rabbit-hole) and [this Medium Post](https://medium.com/dappio-wonderland/the-solution-to-composability-universal-rabbit-hole-28b817cc0fd4) for more details

### Architecture

There are two modules in Universal Rabbit Hole: **Navigator** and **Gateway**:

![](https://hackmd.io/_uploads/rJbWYMd-o.jpg)

#### Navigator (Reader)

Navigator is a Typescript client for instantiating various of kinds of DeFi protocols. You can use it as a standalone dependency in your own project or together with [Dappio Gateway](https://guide.dappio.xyz/the-universal-rabbit-hole).

#### Gateway (Writer)

Gateway is a **CaaS (Compasaility-as-a-Service)** that standardizes inter-protocol interaction on Solana to unlock the potential of composability. It has an universal interface for various Solana DeFi elements (Pool / Farm / MoneyMarket / Vault / Leveraged Farm / ...) and serves as a **common knowledge base** that helps Solana community learn and improve.

### Anatomy of Gateway

![](https://hackmd.io/_uploads/Skbcueoyi.jpg)

Let's take a closer look on Gateway module. There are 4 different components in order to make Gateway function:
  - **Builder**: Off-chain component that helps composing different DeFi actions
  - **Protocol(s)**: Off-chain component that packages the specific instruction set of each protocol
  - **Gateway**: On-chain program that receives and dispatches all the transactions, manages state, distributes fees
  - **Adapter(s)**: On-chain program that connects base program and Gateway program

While **Builder** and **Protocols** are packaged into a npm module, **Gateway (program)** and **Adapters** are Solana programs deployed by Dappio (sometimes third-party) to be invoked to interact with Base programs
