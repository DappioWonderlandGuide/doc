---
tags: docs
---

# Quick Start [WIP]

> See this [repo](https://github.com/DappioWonderland/simple-composer) for the full example

In this quick start script, we are going to demonstrate how to quickly compose `addLiquidity` and `stake` for Saber protocol **in less than 50 lines of code**.

## Setup

#### Create new repo

```bash
$ mkdir my-composer
$ cd my-composer
```

#### Initialize npm package

```bash
$ npm init -y
```

#### Initialize Typescript

```bash
$ yarn global add typescript
$ tsc --init
```

#### Install dependencies

```bash
$ yarn add @project-serum/anchor@^0.24.2
$ yarn add @dappio-wonderland/navigator@^0.2.17
$ yarn add @dappio-wonderland/gateway@^0.2.20
```

## Add a new Composer

Create a new file `index.ts`:

```bash
$ touch index.ts
```

Copy and paste the following code snippet to `index.ts`:

```typescript
import * as anchor from "@project-serum/anchor";
import NodeWallet from "@project-serum/anchor/dist/cjs/nodewallet";
import { AddLiquidityParams, StakeParams, GatewayBuilder, SupportedProtocols } from "@dappio-wonderland/gateway";

// 1-1. Create a RPC connection
const connection = new anchor.web3.Connection("https://rpc-mainnet-fork.epochs.studio", {
  commitment: "confirmed",
  wsEndpoint: "wss://rpc-mainnet-fork.epochs.studio/ws",
});

// 1-2. Setup Anchor provider
const options = anchor.AnchorProvider.defaultOptions();
const wallet = NodeWallet.local();
const provider = new anchor.AnchorProvider(connection, wallet, options);
anchor.setProvider(provider);

// 1-3. Setup Gateway
const gateway = new GatewayBuilder(provider);

const main = async () => {
  // 2-1. Config addLiquidity parameters
  const zapInAmount = 10000;
  const poolId = new anchor.web3.PublicKey(
    "YAkoNb6HKmSxQN9L8hiBE5tPJRsniSSMzND1boHmZxe" // USDC-USDT
  );

  const addLiquidityParams: AddLiquidityParams = {
    protocol: SupportedProtocols.Saber,
    poolId,
    tokenInAmount: zapInAmount,
    tokenMint: new anchor.web3.PublicKey("EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"),
  };

  // 2-2. Config stake parameters
  const farmId = new anchor.web3.PublicKey(
    "Hs1X5YtXwZACueUtS9azZyXFDWVxAMLvm3tttubpK7ph" // USDC-USDT
  );

  const stakeParams: StakeParams = {
    protocol: SupportedProtocols.Saber,
    farmId,
  };

  // 2-3. Compose
  await gateway.addLiquidity(addLiquidityParams);
  await gateway.stake(stakeParams);

  // 3-1. Generate transactions
  await gateway.finalize();
  const txs = gateway.transactions();

  // 3-2. Send all transactions
  console.log("======");
  console.log("Txs are sent...");
  for (let tx of txs) {
    const sig = await provider.sendAndConfirm(tx, [], {
      skipPreflight: true,
      commitment: "confirmed",
    } as unknown as anchor.web3.ConfirmOptions);
    console.log(sig);
  }
  console.log("Txs are executed");
  console.log("======");
};

main();

export {};
```

## Run the Composer

```bash
$ ANCHOR_WALLET=~/.config/solana/gateway.json ts-node index.ts
```

> TODO: Use airdrop
> TODO: Fix Jupiter issue on mf
