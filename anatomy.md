---
tags: docs
---

# Anatomy of Universal Rabbit Hole [WIP]

> TODO: Update diagram

![](https://hackmd.io/_uploads/Skbcueoyi.jpg)

Let's take a closer look on Gateway module. There are 4 different components in order to make Gateway function:
  - **Builder (Gateway Client)**: Off-chain component that helps composing different DeFi actions
  - **Protocol(s)**: Off-chain component that packages the specific instruction set of each protocol
  - **Gateway Program**: On-chain program that receives and dispatches all the transactions, manages state, distributes fees
  - **Adapter Program(s)**: On-chain program that connects base program and Gateway program

While **Builder** and **Protocols** are packaged into a npm module, **Gateway (program)** and **Adapters** are Solana programs deployed by Dappio (sometimes third-party) to be invoked to interact with Base programs
