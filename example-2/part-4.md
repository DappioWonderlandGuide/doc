---
tags: docs
---

# Example 2 - Implement Gateway Client (Part 4) [WIP]

> TODO

- Client side typescript library
- Write functions

Recall how we map adapter program to the base program eariler, now we'll need to generate all accounts the instruction needed and wrap it through gateway. That's take a closer look at `builder.ts`, you might notice almost all functions in `GatewayBuilder` class returned `GatewayBuilder` type, which means user can chain all kinds of DeFi actions together, will see more about how to compose them together in the test script later.

```typescript
export class GatewayBuilder {
  async swap(swapParams: SwapParams): Promise<GatewayBuilder> {
    ...
  }
  async addLiquidity(addLiquidityParams: AddLiquidityParams): Promise<GatewayBuilder> {
    ...
  }
  async removeLiquidity(removeLiquidityParams: RemoveLiquidityParams): Promise<GatewayBuilder> {
    ...
  }
  async stake(stakeParams: StakeParams): Promise<GatewayBuilder> {
    ...
  }
  async unstake(unstakeParams: UnstakeParams): Promise<GatewayBuilder> {
    ...
  }
  async harvest(harvestParams: HarvestParams): Promise<GatewayBuilder> {
    ...
  }
}
```

And now let's start to implement write functions in gateway. Find corresponding functions you have to code in `types.ts`, for Genopests `IProtocolFarm` is the only one interface they needed. The type of interfaces should be instantiated depends on what kind of actions the base program supports.

```typescript
export class ProtocolGenopets implements IProtocolFarm {
  constructor(
    private _connection: anchor.web3.Connection,
    private _gatewayProgram: anchor.Program<Gateway>,
    private _gatewayStateKey: anchor.web3.PublicKey,
    private _gatewayParams: GatewayParams
  ) {}

  async stake(
    params: StakeParams,
    farmInfo: IFarmInfo,
    userKey: anchor.web3.PublicKey
  ): Promise<{ txs: anchor.web3.Transaction[]; input: Buffer }> {
    ...
    return { txs: [txStake], input: payload };
  }

  async unstake(
    params: UnstakeParams,
    farmInfo: IFarmInfo,
    userKey: anchor.web3.PublicKey
  ): Promise<{ txs: anchor.web3.Transaction[]; input: Buffer }> {
    ...
    return { txs: [txUnstake], input: payload };
  }

  async harvest(
    params: HarvestParams,
    farmInfo: IFarmInfo,
    userKey: anchor.web3.PublicKey
  ): Promise<{ txs: anchor.web3.Transaction[]; input: Buffer }> {
    ...
    return { txs: [txHarvest], input: payload };
  }
}
```

After all functions implemented, by adding switch case for all actions you just implemented in `builder.ts`, and it's good to support new protocol in gateway now!

```typescript
case SupportedProtocols.Genopets:
  this._metadata.farm = await genopets.infos.getFarm(
    this._provider.connection,
    stakeParams.farmId
  );
  protocol = new ProtocolGenopets(
    this._provider.connection,
    this._program,
    await this.getGatewayStateKey(),
    this.params
  );

  break;
```
