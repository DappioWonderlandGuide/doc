---
tags: docs
---

# Example 1: Add Tulip Vaults (Part 3) [WIP]

In the final part of the guide, we will demonstrate how simple it is to **compose** 2 different protocols into one continuous operation.

### Update `NavigatorProvider`

Let's start from replacing `NavigatorProvider.tsx` with the following code snippet:

```typescript!
// src/contexts/NavigatorProvider.tsx

import { useConnection } from "@solana/wallet-adapter-react";
import {
  createContext,
  FC,
  ReactNode,
  useContext,
  useEffect,
  useState,
} from "react";
// Added from Part 3
import { raydium, orca, tulip } from "@dappio-wonderland/navigator";

export interface NavigatorContextState {
  raydiumFarms: raydium.FarmInfoWrapper[];
  raydiumPoolSetWithLpMintKey: Map<string, raydium.PoolInfoWrapper>;
  orcaFarms: orca.FarmInfoWrapper[];
  orcaPoolSetWithLpMintKey: Map<string, orca.PoolInfoWrapper>;
  // Added from Part 3
  tulipVaults: tulip.VaultInfoWrapper[];
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
  const [orcaFarms, setOrcaFarms] = useState<orca.FarmInfoWrapper[]>([]);
  const [orcaPoolSetWithLpMintKey, setOrcaPoolSetWithLpMintKey] = useState<
    Map<string, orca.PoolInfoWrapper>
  >({} as Map<string, orca.PoolInfoWrapper>);

  // Added from Part 3
  const [tulipVaults, setTulipVaults] = useState<tulip.VaultInfoWrapper[]>([]);

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
    // *************************
    // Added from Part 3 (Begin)
    {
      const getAllVaultsWrappers = async () => {
        return (await tulip.infos.getAllVaultWrappers(
          connection
        )) as tulip.VaultInfoWrapper[];
      };
  
      getAllVaultsWrappers().then((wrappers) => {
        setTulipVaults(wrappers);
      });
    }
    // Added from Part 3 (End)
    // *************************
  }, []);

  return (
    <NavigatorContext.Provider
      value={{
        raydiumFarms,
        raydiumPoolSetWithLpMintKey,
        orcaFarms,
        orcaPoolSetWithLpMintKey,
        // Added from Part 3
        tulipVaults,
      }}
    >
      {children}
    </NavigatorContext.Provider>
  );
};
```

What we do in above code snippet is fetching vaults from Tulip by **one line of code** (`tulip.infos.getAllVaultWrappers`)

### Add `tulipVaults`

Copy from `raydiumFarms`:

```
$ cp src/pages/raydiumFarms.tsx src/pages/tulipVaults.tsx
```

Replace `tulipVaults.tsx` with the following code snippet:

```typescript!
// src/pages/tulipVaults.tsx

import { useNavigator } from "contexts/NavigatorProvider";
import { NextPage } from "next";
import Head from "next/head";
import { useEffect, useState } from "react";
import { tulip as protocol } from "@dappio-wonderland/navigator";
import { Vault } from "components/TulipVault";

export const TulipVaults: NextPage = (props) => {
  const { tulipVaults, raydiumPoolSetWithLpMintKey } = useNavigator();
  const [vaultsWithPool, setVaultsWithPool] = useState<
    protocol.VaultInfoWrapper[]
  >([]);

  useEffect(() => {
    const vaultsWithPool = tulipVaults.filter((vault) => {
      return raydiumPoolSetWithLpMintKey.size > 0
        ? raydiumPoolSetWithLpMintKey.has(
            vault.vaultInfo.base.underlyingMint.toString()
          )
        : false;
    });
    setVaultsWithPool(vaultsWithPool);
  }, [raydiumPoolSetWithLpMintKey]);

  return (
    <div>
      <Head>
        <title>Solana Scaffold</title>
        <meta name="description" content="Farms" />
      </Head>
      <div className="md:hero mx-auto p-4">
        <div className="md:hero-content flex flex-col">
          <h1 className="text-center text-5xl font-bold text-transparent bg-clip-text bg-gradient-to-tr from-[#9945FF] to-[#14F195]">
            Tulip Vaults
          </h1>
          {/* CONTENT GOES HERE */}
          <div className="overflow-x-auto">
            <table className="table w-full">
              <thead>
                <tr>
                  <th>Vault ID</th>
                  <th>LP Token</th>
                  <th></th>
                </tr>
              </thead>
              <tbody>
                {vaultsWithPool
                  .sort((a, b) =>
                    a.vaultInfo.vaultId
                      .toString()
                      .localeCompare(b.vaultInfo.vaultId.toString())
                  )
                  .map((vault) => (
                    <Vault
                      key={vault.vaultInfo.vaultId.toString()}
                      vault={vault}
                      pool={raydiumPoolSetWithLpMintKey.get(
                        vault.vaultInfo.base.underlyingMint.toString()
                      )}
                    ></Vault>
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

export default TulipVaults;
```

Since we're going to implement deposit to and withdraw from Tulip vualt, `farm` will no longer be used.

As a result:
- We replace `RaydiumFarm` to `TulipVault`, `raydiumFarm` to `tulipVault`, `Farm` to `Vault`, and `farm` to `vault`.
- We change the properties from `poolLpTokenAccount.mint` to `base.underlyingMint`.

### Add `TulipVault`

Copy from `RaydiumFarm`:

```
$ cp src/components/RaydiumFarm.tsx src/components/TulipVault.tsx
```

Replace `TulipVault.tsx` with the following code snippet:

```typescript!
// src/components/TulipVault.tsx

import { FC, useCallback } from "react";
import { PublicKey } from "@solana/web3.js";
import { notify } from "utils/notifications";
import { useConnection, useWallet } from "@solana/wallet-adapter-react";
import useUserSOLBalanceStore from "stores/useUserSOLBalanceStore";
import { AnchorWallet } from "utils/anchorWallet";
import * as anchor from "@project-serum/anchor";
import { raydium, tulip } from "@dappio-wonderland/navigator";
import {
  AddLiquidityParams,
  GatewayBuilder,
  RemoveLiquidityParams,
  DepositParams,
  SupportedProtocols,
  SwapParams,
  WithdrawParams,
  WSOL,
} from "@dappio-wonderland/gateway";

interface VaultProps {
  vault: tulip.VaultInfoWrapper;
  pool: raydium.PoolInfoWrapper;
}

export const Vault: FC<VaultProps> = (props: VaultProps) => {
  const { connection } = useConnection();
  const wallet = useWallet();
  const { getUserSOLBalance } = useUserSOLBalanceStore();

  // Get Farm
  const vault = props.vault;
  const vaultInfo = vault.vaultInfo;
  const vaultId = vaultInfo.vaultId.toString();
  const lpMint = vaultInfo.base.underlyingMint.toString();
  const pool = props.pool;
  const poolInfo = pool.poolInfo;

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
      protocol: SupportedProtocols.Raydium,
      poolId: poolInfo.poolId,
    };
    const depositParams: DepositParams = {
      protocol: SupportedProtocols.Tulip,
      vaultId: vaultInfo.vaultId,
      depositAmount: 0, // Notice: amount will auto-update in gateway state after add liquidity
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
    await gateway.deposit(depositParams);

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
          skipPreflight: false,
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

    const depositorId = tulip.infos.getDepositorId(
      vaultInfo.vaultId,
      wallet.publicKey
    );
    const depositor = (await tulip.infos.getDepositor(
      connection,
      depositorId
    )) as tulip.DepositorInfo;

    const shareAmount = Math.floor(Number(depositor.shares) / 10);

    const withdrawParams: WithdrawParams = {
      protocol: SupportedProtocols.Tulip,
      vaultId: vaultInfo.vaultId,
      withdrawAmount: shareAmount,
    };
    const removeLiquidityParams: RemoveLiquidityParams = {
      protocol: SupportedProtocols.Raydium,
      poolId: poolInfo.poolId,
    };

    const { tokenAAmount: coinAmount } = pool.getTokenAmounts(shareAmount);
    // tokenA to tokenB
    const swapParams1: SwapParams = {
      protocol: SupportedProtocols.Jupiter,
      fromTokenMint: poolInfo.tokenAMint,
      toTokenMint: poolInfo.tokenBMint,
      amount: coinAmount, // swap coin to pc
      slippage: 3,
    };

    // tokenB to WSOL
    const swapParams2: SwapParams = {
      protocol: SupportedProtocols.Jupiter,
      fromTokenMint: poolInfo.tokenBMint,
      toTokenMint: new PublicKey(WSOL),
      amount: 0, // Notice: This amount needs to be updated later
      slippage: 3,
    };

    const gateway = new GatewayBuilder(provider);

    await gateway.withdraw(withdrawParams);
    await gateway.removeLiquidity(removeLiquidityParams);

    // 1st Swap
    await gateway.swap(swapParams1);
    const minOutAmount = gateway.params.swapMinOutAmount.toNumber();
    swapParams2.amount = minOutAmount;

    // 2nd Swap
    if (!poolInfo.tokenBMint.equals(WSOL)) {
      await gateway.swap(swapParams2);
    }

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
          skipPreflight: false,
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
        {vaultId.slice(0, 5)}...{vaultId.slice(vaultId.length - 5)}
      </th>
      <td>
        {lpMint.slice(0, 5)}...{lpMint.slice(lpMint.length - 5)}
      </td>
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

You will be surprised by the extreme simplicity of the interface. **To compose 2 actions from 2 protocols, the only thing you have to do is specifying the protocol type and action.** That's it!

```typescript!
const gateway = new GatewayBuilder(provider);

await gateway.swap(swapParams1);

// ...

await gateway.swap(swapParams2);

// ...

await gateway.addLiquidity(addLiquidityParams);

// ...

await gateway.deposit(depositParams);

await gateway.finalize();
const txs = gateway.transactions();
```

### Update `ContentContainer`

Let's add a link for `tulipVaults`:

```typescript!
// src/components/ContentContainer.tsx 

// #L39
<li>
  <Link href="/tulipVaults">
    <a>Tulip Vaults</a>
  </Link>
</li>
```

### Compile and Run

```bash
$ yarn dev
```

Then open http://localhost:3000 (default).
