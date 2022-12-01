---
tags: docs
---

# Workflow [WIP]

> TODO

![](https://hackmd.io/_uploads/rkZ6gRbSo.png)

We can separate these series of operations into 2 distinct sections: **off-chain part** and **on-chain part**.

### Off-chain Part

- User assembles actions by invoking action setter and providing proper parameters
- Builder composes transactions

### On-chain Part

- User sends transactions directly to Gateway program
- Gateway dispatches transactions to their corresponding adapters through CPI (cross-program invocation)
- Adapter invokes Base program through another CPI
