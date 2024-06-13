## Build the Source Code

We’re going to be spinning up an EVM Rollup from the OP Stack source code.  We can use docker images as well, but this way we can modify component behavior if we need to do so. The OP Stack source code is split between two repositories, the [Optimism Monorepo](https://github.com/ethereum-optimism/optimism) and the [`op-geth`](https://github.com/ethereum-optimism/op-geth) repository.

### Build the Optimism Monorepo

1. Clone the [Optimism Monorepo](https://github.com/ethereum-optimism/optimism).

    ```bash
    cd ~
    git clone https://github.com/ethereum-optimism/optimism.git
    ```

2. Enter the Optimism Monorepo.

    ```bash
    cd optimism
    ```

3. Install required modules. This is a slow process, while it is running you can already start building `op-geth`, as shown below.

    ```bash
    pnpm install
    ```

4. Build the various packages inside of the Optimism Monorepo.

    ```bash
    make op-node op-batcher op-proposer
    pnpm build
    ```

    Please note that this a considerably longer step (about 15 mins) and §solc can take up 

### Build op-geth

1. Clone [`op-geth`](https://github.com/ethereum-optimism/op-geth):

    ```bash
    cd ~
    git clone https://github.com/ethereum-optimism/op-geth.git
    ```

1. Enter `op-geth`:

    ```bash
    cd op-geth
    ```

1. Build `op-geth`:

    ```bash
    make geth
    ```

## Get access to a Holesky node

Since we’re deploying our OP Stack chain to Goerli, we’ll need to have access to a Goerli L1 node. We can either use a node provider like [Alchemy](https://www.alchemy.com/) (easier) or [run your own Holesky node](https://notes.ethereum.org/@launchpad/holesky) (harder).

## Generate some keys

We’ll need four accounts and their private keys when setting up the chain:

- The `Admin` account which has the ability to upgrade contracts.
- The `Batcher` account which publishes Sequencer transaction data to L1.
- The `Proposer` account which publishes L2 transaction results to L1.
- The `Sequencer` account which signs blocks on the p2p network.

You can generate all of these keys with the `rekey` tool in the `contracts-bedrock` package.


1. Enter the Optimism Monorepo:

    ```bash
    cd ~
    cd optimism
    ```

1. Move into the `contracts-bedrock` package:

    ```bash
    cd packages/contracts-bedrock
    ```

1. Use `cast wallet` to generate new accounts

    ```bash
    echo "Admin:"
    cast wallet new
    echo "Proposer:"
    cast wallet new
    echo "Batcher:"
    cast wallet new
    echo "Sequencer:"
    cast wallet new
    ```

We get an output like the following:

```Admin:
Successfully created new keypair.
Address:     0xC4481aa21AeAcAD3cCFe6252c6fe2f161A47A771
Private key: 0x3dd6681bda6458773e9e8aa6b45574b73a953eaea67ff6d9d00795da73e39177
Proposer:
Successfully created new keypair.
Address:     0x0F10759D26F07a5D9e7F4BC0b54d1e4CAAEd15C9
Private key: 0x723fcb35ce979dea9f269c26d0cd685cfd45cd90681900c1e69fa9c9ba0479d8
Batcher:
Successfully created new keypair.
Address:     0xab110dA2064AC0B44c08D71A3D8148BBB0C3aD1F
Private key: 0x2e9c494f5a6c0325ea5bd2265132f73dd040a6d75bd4303866c10ff1ec51b356
Sequencer:
Successfully created new keypair.
Address:     0x50987039A15c83C4090eD5ecfda9E7F07160D4a0
Private key: 0x2c65361c17805b0b8da483cbc7fd462608f1c3fd73f4ac5e809082a2c52cf879```

I'll save these accounts and their respective private keys somewhere for now. Fund the `Admin` address with a small amount of Holesky ETH as we’ll use that account to deploy our smart contracts. You’ll also need to fund the `Proposer` and `Batcher` address — note that the `Batcher` burns through the most ETH because it publishes transaction data to L1.

Recommended funding amounts are as follows:

- `Admin` — 2 ETH
- `Proposer` — 5 ETH
- `Batcher` — 10 ETH

::: danger Not for production deployments

The `cast wallet new` tool is *not* designed for production deployments. If we are deploying an OP Stack based chain into production, we should likely be using a combination of hardware security modules and hardware wallets.

:::