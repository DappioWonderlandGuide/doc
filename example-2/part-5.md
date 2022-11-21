---
tags: docs
---

# Example 2 - Write Test (Part 5) [WIP]

- Compose!
- **Swap (Jupiter) + AddLiquidity (Raydium) + Stake (Genopets)** in less than 10 lines of code

How to use gateway to enjoy the benefits of composibility? **3 simple steps can make magic happend:**

```typescript
// 1. Initialize GatewayBuilder
const gateway = new GatewayBuilder(provider);

// 2. setup whatever DeFi action you want
const swapParams: SwapParams = {
  protocol: SupportedProtocols.Jupiter,
  fromTokenMint: new PublicKey(
    "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v" // USDC
  ),
  toTokenMint: new PublicKey(
    "GENEtH5amGSi8kHAtQoezp1XEXwZJ8vcuePYnXdKrMYz" // GENE
  ),
  amount: 100, 
  slippage: 1,
};
await gateway.swap(swapParams);

const addLiquidityParams: AddLiquidityParams = {
  protocol: SupportedProtocols.Raydium,
  poolId,
  tokenInAmount: 100,
};
await gateway.addLiquidity(addLiquidityParams);

const stakeParams: StakeParams = {
  protocol: SupportedProtocols.Genopets,
  farmId,
  version: 1,
  lpAmount: 10000,
  lockDuration: 0,
  mint,
};
await gateway.stake(stakeParams);

// 3. Finalize to get all txs!
await gateway.finalize();
const txs = gateway.transactions();
```
