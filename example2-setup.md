---
tags: docs
---

# Example 2 - Setup [WIP]

### Env

- Rust `1.64.0`
- Cargo `1.64.0`
- Solana-cli `1.10.32`
- rustup toolchain `stable`
- anchor `0.24.2`
- node `17.3.0`

### Run the Test

#### Config Private Key

```
$ touch ~/.config/solana/gateway.json
```

and copy and paste the test private key to `gateway.json`

#### Run script

```
$ git clone git@github.com:DappioWonderland/gateway.git
```

```
$ yarn run testAdapterRaydium
...
```

```
$ yarn run testAdapterSaber
...
```
