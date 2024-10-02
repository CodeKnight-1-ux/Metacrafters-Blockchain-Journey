### Overview

The [`tokenvm`](./examples/tokenvm) was developed as an example to demonstrate how to integrate the `hypersdk` for token-related functionalities such as minting and trading. With the `tokenvm`, users can issue their own assets, mint additional units, update asset metadata, and burn tokens. Additionally, there is a built-in decentralized exchange mechanism that allows users to create and fulfill trade orders with any token in circulation. This implementation also includes a command-line interface (CLI) and supports RPC-based trade requests using an in-memory order book that synchronizes block data in real-time.

If you're interested in blockchain-based exchanges, `tokenvm` provides a simple yet efficient codebase (with order-filling logic under 100 lines of code) to explore.

### Current Status

The `tokenvm` is currently **ALPHA** software. It is still in the experimental phase and should not be used in production environments. The framework is actively evolving and is expected to undergo significant changes as modules are refined and security is audited.

### Key Features

#### Custom Token Minting

At its core, `tokenvm` allows users to create, mint, and transfer custom tokens. When an asset is created, the issuer holds administrative control, enabling them to mint more tokens, update metadata (such as adding new information), and transfer or revoke ownership. 

The storage system is optimized for this token management, with each balance requiring just 72 bytes of state (`assetID|publicKey=>balance(uint64)`), making transfers efficient and allowing parallel processing of transactions that do not involve the same accounts. Once upstream optimizations in `hypersdk` are completed, this parallelism will be automatically applied to the `tokenvm`.

#### Token Trading

Token creation is just the beginning; `tokenvm` also supports full on-chain trading. Users can create offers for specific assets, defining the rate and quantity they are willing to exchange, and other participants can accept and fill these orders. `tokenvm` maintains an in-memory order book, accessible via RPC, which lists these active orders.

Like balances, orders are optimized for storage, each requiring only 152 bytes of state (`orderID=>inAsset|inTick|outAsset|outTick|remaining|owner`). This design allows for efficient parallel execution of trades, even within the same trading pair.

##### In-Memory Order Book

To enhance user experience, `tokenvm` includes an in-memory order book that tracks active orders for specified trading pairs. This uses `hypersdk`'s transaction monitoring to ensure that the order book reflects the most up-to-date on-chain data.

##### Protection from Front-Running

Unlike some other decentralized trading platforms, `tokenvm` requires the user to specify the exact order they want to interact with. This means bots or malicious actors cannot manipulate the price by front-running your transaction. The worst outcome is that they reduce the available quantity of the asset before you complete your trade.

##### Partial Fills and Refunds

Users don’t need to fulfill entire orders; partial fills are supported. If a user overfills an order, any excess input is automatically refunded, ensuring that no funds are lost due to blockchain transaction timing.

##### Time-Limited Orders

When placing an order, users can set a time limit, ensuring that the trade is only valid for a specified duration. This protects traders from unwanted long-standing offers remaining open until explicitly canceled.

#### Avalanche Warp Messaging (AWM) Integration

`tokenvm` utilizes Avalanche Warp Messaging (AWM), allowing users to transfer assets between `tokenvm` instances on different subnets without relying on third-party bridges. A minimum of 80% validator consensus is required for cross-chain messages, ensuring security. Assets transferred from other `tokenvm` instances are tracked by unique `AssetIDs` to prevent manipulation by rogue subnets.

To avoid cascading failures, only natively minted assets can be exported to another `tokenvm`, ensuring transparent asset policies and preventing cross-chain asset vulnerabilities.

Users can also tip message relayers and swap assets upon receipt to cover transaction fees, making the cross-subnet transfers seamless.

For further details on the implementation, refer to the [E2E test suite](./tests/e2e/e2e_test.go).

### Demonstrations

If you're looking to explore `tokenvm` in action, you can start by launching your own `tokenvm` subnet. Here's how:

```bash
./scripts/run.sh
```

By default, the network allocates all funds to the address `token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp`. The private key is stored in the `demo.pk` file for convenience.

If you only need one subnet for testing, you can run:

```bash
MODE="run-single" ./scripts/run.sh
```

#### Building the CLI

To interact with `tokenvm`, build the command-line interface with the following command:

```bash
./scripts/build.sh
```

The CLI will be located in the `./build/token-cli` directory.

#### Adding Chains and Keys

Next, import the default key and add the chain using these commands:

```bash
./build/token-cli key import demo.pk
./build/token-cli chain import-anr
```

This connects to the Avalanche Network Runner and retrieves node URIs for all chains created.

### Example Use Cases

#### Creating an Asset

To create a new asset:

```bash
./build/token-cli action create-asset
```

The output will show your new asset's details, including its `txID` which serves as its `assetID`.

#### Minting an Asset

After creating an asset, you can mint tokens by running:

```bash
./build/token-cli action mint-asset
```

#### Checking Balance

To confirm the minted tokens, check your balance:

```bash
./build/token-cli key balance
```
