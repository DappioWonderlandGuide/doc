---
tags: docs
---

# Tutorial 1 - Add Orca Farms (Part 2) [WIP]

In this part, we will continue to add Orca Farms. You will soon discover that **the interfaces are exactly identical no matter which protocol you are trying to access.**

### Update `NavigatorProvider`

Replace `NavigatorPrivider.tsx` with the following code snippet:

```typescript!
// src/contexts/NavigatorPrivider.tsx

import { useConnection } from "@solana/wallet-adapter-react";
import {
  createContext,
  FC,
  ReactNode,
  useContext,
  useEffect,
  useState,
} from "react";
// Added from Part 2
import { raydium, orca } from "@dappio-wonderland/navigator";

export interface NavigatorContextState {
  raydiumFarms: raydium.FarmInfoWrapper[];
  raydiumPoolSetWithLpMintKey: Map<string, raydium.PoolInfoWrapper>;
  // *************************
  // Added from Part 2 (Begin)
  orcaFarms: orca.FarmInfoWrapper[];
  orcaPoolSetWithLpMintKey: Map<string, orca.PoolInfoWrapper>;
  // Added from Part 2 (End)
  // *************************
}

export const NavigatorContext = createContext<NavigatorContextState>(
  {} as NavigatorContextState
);

export function useNavigator(): NavigatorContextState {
  return useContext(NavigatorContext);
}

export const NavigatorProvider: FC<{ children: ReactNode }> = ({
  children,
}) => {
  const { connection } = useConnection();
  const [raydiumFarms, setRaydiumFarms] = useState<raydium.FarmInfoWrapper[]>(
    []
  );
  const [raydiumPoolSetWithLpMintKey, setRaydiumPoolSetWithLpMintKey] =
    useState<Map<string, raydium.PoolInfoWrapper>>(
      {} as Map<string, raydium.PoolInfoWrapper>
    );
  // *************************
  // Added from Part 2 (Begin)
  const [orcaFarms, setOrcaFarms] = useState<orca.FarmInfoWrapper[]>([]);
  const [orcaPoolSetWithLpMintKey, setOrcaPoolSetWithLpMintKey] = useState<
    Map<string, orca.PoolInfoWrapper>
  >({} as Map<string, orca.PoolInfoWrapper>);
  // Added from Part 2 (End)
  // *************************
  
  useEffect(() => {
    {
      const getAllFarmsWrappers = async () => {
        return (await raydium.infos.getAllFarmWrappers(
          connection
        )) as raydium.FarmInfoWrapper[];
      };

      getAllFarmsWrappers().then((wrappers) => {
        setRaydiumFarms(wrappers);
      });

      const getAllPoolWrappers = async () => {
        const poolWrappers = await raydium.infos.getAllPoolWrappers(connection);

        return new Map<string, raydium.PoolInfoWrapper>(
          poolWrappers.map((wrapper) => [
            wrapper.poolInfo.lpMint.toString(),
            wrapper as raydium.PoolInfoWrapper,
          ])
        );
      };

      getAllPoolWrappers().then((poolSetResult) => {
        setRaydiumPoolSetWithLpMintKey(poolSetResult);
      });
    }
    // *************************
    // Added from Part 2 (Begin)
    {
      const getAllFarmsWrappers = async () => {
        return (await orca.infos.getAllFarmWrappers(
          connection
        )) as orca.FarmInfoWrapper[];
      };

      getAllFarmsWrappers().then((wrappers) => {
        setOrcaFarms(wrappers);
      });

      const getAllPoolWrappers = async () => {
        const poolWrappers = await orca.infos.getAllPoolWrappers(connection);

        return new Map<string, orca.PoolInfoWrapper>(
          poolWrappers.map((wrapper) => [
            wrapper.poolInfo.lpMint.toString(),
            wrapper as orca.PoolInfoWrapper,
          ])
        );
      };

      getAllPoolWrappers().then((poolSetResult) => {
        setOrcaPoolSetWithLpMintKey(poolSetResult);
      });
    }
    // Added from Part 2 (End)
    // *************************
  }, []);

  return (
    <NavigatorContext.Provider
      value={{
        raydiumFarms,
        raydiumPoolSetWithLpMintKey,
        // *************************
        // Added from Part 2 (Begin)
        orcaFarms,
        orcaPoolSetWithLpMintKey,
        // Added from Part 2 (End)
        // *************************
      }}
    >
      {children}
    </NavigatorContext.Provider>
  );
};
```

Notice the highlighting comments in the code snippet above.

### Add `orcaFarms`

Copy from `raydiumFarms`:

```bash!
$ cp src/pages/raydiumFarms.tsx src/pages/orcaFarms.tsx
```

Replace `ocraFarms.tsx` with the following code snippet:

```typescript!
// src/pages/orcaFarms.tsx

import { useNavigator } from "contexts/NavigatorProvider";
import { NextPage } from "next";
import Head from "next/head";
import { useEffect, useState } from "react";
import { orca } from "@dappio-wonderland/navigator";
import { Farm } from "../components/OrcaFarm";

export const OrcaFarms: NextPage = (props) => {
  const { orcaFarms, orcaPoolSetWithLpMintKey } = useNavigator();
  const [farmsWithPool, setFarmsWithPool] = useState<orca.FarmInfoWrapper[]>(
    []
  );

  useEffect(() => {
    setFarmsWithPool(
      orcaFarms.filter((farm) => {
        return orcaPoolSetWithLpMintKey.size > 0
          ? orcaPoolSetWithLpMintKey.has(farm.farmInfo.baseTokenMint.toString())
          : false;
      })
    );
  }, [orcaFarms]);

  return (
    <div>
      <Head>
        <title>Solana Scaffold</title>
        <meta name="description" content="Farms" />
      </Head>
      <div className="md:hero mx-auto p-4">
        <div className="md:hero-content flex flex-col">
          <h1 className="text-center text-5xl font-bold text-transparent bg-clip-text bg-gradient-to-tr from-[#9945FF] to-[#14F195]">
            Orca Farms
          </h1>
          {/* CONTENT GOES HERE */}
          <div className="overflow-x-auto">
            <table className="table w-full">
              <thead>
                <tr>
                  <th>Farm ID</th>
                  <th>LP Token</th>
                  <th>APY</th>
                  <th></th>
                </tr>
              </thead>
              <tbody>
                {farmsWithPool
                  .sort((a, b) =>
                    a.farmInfo.farmId
                      .toString()
                      .localeCompare(b.farmInfo.farmId.toString())
                  )
                  .map((farm) => (
                    <Farm
                      key={farm.farmInfo.farmId.toString()}
                      farm={farm}
                      pool={orcaPoolSetWithLpMintKey.get(
                        farm.farmInfo.baseTokenMint.toString()
                      )}
                    ></Farm>
                  ))}
              </tbody>
            </table>
          </div>
          <div className="text-center"></div>
        </div>
      </div>
    </div>
  );
};

export default OrcaFarms;
```

Except for the difference in protocol name (Raydium v.s Orca), the interface is almost identical. This means that **you can access all the protocols supported by Universal Rabbit Hole with the same universal interface!**

### Add `OrcaFarm`

Copy from `RaydiumFarm` component:

```bash!
$ cp src/components/RaydiumFarm.tsx src/components/OrcaFarm.tsx
```

Replace if with the following code snippet:

```typescript!
// src/components/OrcaFarm.tsx

import { FC, useCallback, useEffect, useState } from "react";
import { PublicKey } from "@solana/web3.js";
import { notify } from "utils/notifications";
import { useConnection, useWallet } from "@solana/wallet-adapter-react";
import useUserSOLBalanceStore from "stores/useUserSOLBalanceStore";
import { AnchorWallet } from "utils/anchorWallet";
import * as anchor from "@project-serum/anchor";
import { orca as protocol } from "@dappio-wonderland/navigator";
import {
  AddLiquidityParams,
  GatewayBuilder,
  HarvestParams,
  RemoveLiquidityParams,
  StakeParams,
  SupportedProtocols,
  SwapParams,
  UnstakeParams,
  WSOL,
} from "@dappio-wonderland/gateway";

interface FarmProps {
  farm: protocol.FarmInfoWrapper;
  pool: protocol.PoolInfoWrapper;
}

const protocolType = SupportedProtocols.Orca;

export const Farm: FC<FarmProps> = (props: FarmProps) => {
  const [apr, setApr] = useState(0);
  const { connection } = useConnection();
  const wallet = useWallet();
  const { getUserSOLBalance } = useUserSOLBalanceStore();

  // Get Farm
  const farm = props.farm;
  const farmInfo = farm.farmInfo;
  const farmId = farmInfo.farmId.toString();
  const lpMint = farmInfo.baseTokenMint.toString();
  const pool = props.pool;
  const poolInfo = pool.poolInfo;
  useEffect(() => {
    // NOTICE: We mocked LP price and reward price here just for demo
    const aprs = farm.getAprs(5, 1, 2);
    const apr = aprs.length > 1 ? aprs[1] : aprs[0];
    setApr(apr);
  }, []);

  const zapIn = useCallback(async () => {
    if (!wallet.publicKey) {
      console.error("error", "Wallet not connected!");
      notify({
        type: "error",
        message: "error",
        description: "Wallet not connected!",
      });
      return;
    }

    const provider = new anchor.AnchorProvider(
      connection,
      new AnchorWallet(wallet),
      anchor.AnchorProvider.defaultOptions()
    );
    const zapInAmount = 10000; // WSOL Amount

    // WSOL to tokenA
    const swapParams1: SwapParams = {
      protocol: SupportedProtocols.Jupiter,
      fromTokenMint: new PublicKey(WSOL),
      toTokenMint: poolInfo.tokenAMint,
      amount: zapInAmount,
      slippage: 1,
    };
    // tokenA to tokenB
    const swapParams2: SwapParams = {
      protocol: SupportedProtocols.Jupiter,
      fromTokenMint: poolInfo.tokenAMint,
      toTokenMint: poolInfo.tokenBMint,
      amount: 0, // Notice: amount needs to be updated later
      slippage: 1,
    };
    const addLiquidityParams: AddLiquidityParams = {
      protocol: protocolType,
      poolId: poolInfo.poolId,
    };
    const stakeParams: StakeParams = {
      protocol: protocolType,
      farmId: farmInfo.farmId,
    };

    const gateway = new GatewayBuilder(provider);

    // 1st Swap
    await gateway.swap(swapParams1);
    const minOutAmount1 = gateway.params.swapMinOutAmount.toNumber();

    // 2nd Swap
    swapParams2.amount = minOutAmount1 / 2;
    await gateway.swap(swapParams2);
    const minOutAmount2 = gateway.params.swapMinOutAmount.toNumber();

    // Add Liquidity
    addLiquidityParams.tokenInAmount = minOutAmount2;
    await gateway.addLiquidity(addLiquidityParams);

    // Stake
    await gateway.stake(stakeParams);

    await gateway.finalize();
    const txs = gateway.transactions();

    const recentBlockhash = (await connection.getLatestBlockhash()).blockhash;

    txs.forEach((tx) => {
      tx.recentBlockhash = recentBlockhash;
      tx.feePayer = wallet.publicKey;
    });

    const signTxs = await provider.wallet.signAllTransactions(txs);

    console.log("======");
    console.log("Txs are sent...");
    for (let tx of signTxs) {
      let sig: string = "";
      try {
        sig = await connection.sendRawTransaction(tx.serialize(), {
          skipPreflight: true,
          commitment: "confirmed",
        } as unknown as anchor.web3.SendOptions);
        await connection.confirmTransaction(sig, connection.commitment);

        notify({
          type: "success",
          message: "Transaction is executed successfully!",
          txid: sig,
        });
      } catch (error: any) {
        notify({
          type: "error",
          message: `Transaction failed!`,
          description: error?.message,
          txid: sig,
        });
        console.log(
          "NOTICE: paste the output to Transaction Inspector in Solana Explorer for debugging"
        );
        console.log(tx.serializeMessage().toString("base64"));
        console.error("error", `Transaction failed! ${error?.message}`, sig);
        break;
      }
    }
    console.log("Txs are executed");
    console.log("======");

    getUserSOLBalance(wallet.publicKey, connection);
  }, [wallet.publicKey, connection, getUserSOLBalance]);

  const zapOut = useCallback(async () => {
    if (!wallet.publicKey) {
      console.error("error", "Wallet not connected!");
      notify({
        type: "error",
        message: "error",
        description: "Wallet not connected!",
      });
      return;
    }

    const provider = new anchor.AnchorProvider(
      connection,
      new AnchorWallet(wallet),
      anchor.AnchorProvider.defaultOptions()
    );

    // Get share amount
    const ledgerKey = await protocol.infos.getFarmerId(
      farmInfo,
      provider.wallet.publicKey
    );
    const ledger = (await protocol.infos.getFarmer(
      connection,
      ledgerKey
    )) as protocol.FarmerInfo;
    const shareAmount = ledger.amount;
    const { tokenAAmount, tokenBAmount } = await pool.getTokenAmounts(
      shareAmount
    );
    const harvestParams: HarvestParams = {
      protocol: protocolType,
      farmId: farmInfo.farmId,
    };
    const unstakeParams: UnstakeParams = {
      protocol: protocolType,
      farmId: farmInfo.farmId,
      shareAmount,
    };
    const removeLiquidityParams: RemoveLiquidityParams = {
      protocol: protocolType,
      poolId: poolInfo.poolId,
    };
    // tokenB to tokenA
    const swapParams1: SwapParams = {
      protocol: SupportedProtocols.Jupiter,
      fromTokenMint: poolInfo.tokenBMint,
      toTokenMint: poolInfo.tokenAMint,
      amount: tokenBAmount, // swap coin to pc
      slippage: 3,
    };

    // tokenA to WSOL
    const swapParams2: SwapParams = {
      protocol: SupportedProtocols.Jupiter,
      fromTokenMint: poolInfo.tokenAMint,
      toTokenMint: new PublicKey(WSOL),
      amount: 0, // Notice: This amount needs to be updated later
      slippage: 10,
    };

    const gateway = new GatewayBuilder(provider);

    await gateway.harvest(harvestParams);
    await gateway.unstake(unstakeParams);
    await gateway.removeLiquidity(removeLiquidityParams);

    // 1st Swap
    await gateway.swap(swapParams1);
    const minOutAmount = gateway.params.swapMinOutAmount.toNumber();
    swapParams2.amount = minOutAmount + tokenAAmount;
    // 2nd Swap
    await gateway.swap(swapParams2);

    await gateway.finalize();
    const txs = gateway.transactions();

    const recentBlockhash = (await connection.getLatestBlockhash()).blockhash;

    txs.forEach((tx) => {
      tx.recentBlockhash = recentBlockhash;
      tx.feePayer = wallet.publicKey;
    });

    const signTxs = await provider.wallet.signAllTransactions(txs);

    console.log("======");
    console.log("Txs are sent...");
    for (let tx of signTxs) {
      let sig: string = "";
      try {
        sig = await connection.sendRawTransaction(tx.serialize(), {
          skipPreflight: true,
          commitment: "confirmed",
        } as unknown as anchor.web3.SendOptions);
        await connection.confirmTransaction(sig, connection.commitment);

        notify({
          type: "success",
          message: "Transaction is executed successfully!",
          txid: sig,
        });
      } catch (error: any) {
        notify({
          type: "error",
          message: `Transaction failed!`,
          description: error?.message,
          txid: sig,
        });
        console.log(
          "NOTICE: paste the output to Transaction Inspector in Solana Explorer for debugging"
        );
        console.log(tx.serializeMessage().toString("base64"));
        console.error("error", `Transaction failed! ${error?.message}`, sig);
        break;
      }
    }
    console.log("Txs are executed");
    console.log("======");

    getUserSOLBalance(wallet.publicKey, connection);
  }, [wallet.publicKey, connection, getUserSOLBalance]);

  return (
    <tr>
      <th>
        {farmId.slice(0, 5)}...{farmId.slice(farmId.length - 5)}
      </th>
      <td>
        {lpMint.slice(0, 5)}...{lpMint.slice(lpMint.length - 5)}
      </td>
      <td>{apr.toFixed(2) + "%"}</td>
      <td>
        <button className="btn btn-info" onClick={zapIn}>
          Zap In
        </button>
        &nbsp;
        <button className="btn btn-warning" onClick={zapOut}>
          Zap Out
        </button>
      </td>
    </tr>
  );
};
```

Here, you can still feel the power of Gateway: The only change you need to make, is changing protocol type from `Raydium` to `Orca`! The rest of the code are almost identical.

### Update `ContentContainer`

Let's add another link for `orcaFarms`:

```typescript!
// src/components/ContentContainer.tsx 

// #L32
<li>
  <Link href="/orcaFarms">
    <a>Orca Farms</a>
  </Link>
</li>
```

### Compile and Run

```
$ yarn dev
```

Then open http://localhost:3000 (default).
