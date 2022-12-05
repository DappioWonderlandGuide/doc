---
tags: docs
---

# Tutorial 2 - Implement Navigator (Part 3) [WIP]

> TODO

> Here, we actually skip the gateway-program update part.

- Client side typescript library
- Read functions

First, we'll need to know what functions should be implemented. In `types.ts` we define some common interface for similar action. Take Genopets as an example, they allow user **STAKE** $GENE or GENE-USDC in their staking pool to **EARN** $GENE or $SGENE, this is a kind of farming process, so they'll need to implement `Farm` interface.

```typescript
export interface IInstanceFarm {
  getAllFarms(connection: Connection, rewardMint?: PublicKey): Promise<IFarmInfo[]>;
  getAllFarmWrappers(connection: Connection, rewardMint?: PublicKey): Promise<IFarmInfoWrapper[]>;
  getFarm(connection: Connection, farmId: PublicKey): Promise<IFarmInfo>;
  getFarmWrapper(connection: Connection, farmId: PublicKey): Promise<IFarmInfoWrapper>;
  parseFarm(data: Buffer, farmId: PublicKey): IFarmInfo;
  getAllFarmers(connection: Connection, userKey: PublicKey): Promise<IFarmerInfo[]>;
  getFarmerId(farmInfo: IFarmInfo, userKey: PublicKey, version?: number): Promise<PublicKey>;
  getFarmer(connection: Connection, farmerId: PublicKey, version?: number): Promise<IFarmerInfo>;

  // Optional Methods
  getFarmerIdWithBump?(farmId: PublicKey, userKey: PublicKey): [PublicKey, number];
}
```
During implementation, there're two things need to be aware of
1. `*Info` must cantain all essenstial data for write function
(Combine multiple accounts if needed)
2. Try your best to reduce RPC request

By following these two rules, user can easily get all the infos they need within one function and also save RPC endpoint's resource.
