## Build the Source Code

We’re going to be spinning up an EVM Rollup from the OP Stack source code.  We can use docker images as well, but this way we can modify component behavior if we need to do so. The OP Stack source code is split between two repositories, the [Optimism Monorepo](https://github.com/ethereum-optimism/optimism) and the [`op-geth`](https://github.com/ethereum-optimism/op-geth) repository.

### Build the Optimism Monorepo

1. Clone the [Optimism Monorepo](https://github.com/ethereum-optimism/optimism).

    ```bash
    cd ~
    git clone https://github.com/ethereum-optimism/optimism.git
    ```

1. Enter the Optimism Monorepo.

    ```bash
    cd optimism
    ```

1. Install required modules. This is a slow process, while it is running you can already start building `op-geth`, as shown below.

    ```bash
    pnpm install
    ```

1. Build the various packages inside of the Optimism Monorepo.

    ```bash
    make op-node op-batcher op-proposer
    pnpm build
    ```

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

## Get access to a Goerli node

Since we’re deploying our OP Stack chain to Goerli, you’ll need to have access to a Goerli L1 node. You can either use a node provider like [Alchemy](https://www.alchemy.com/) (easier) or [run your own Goerli node](https://notes.ethereum.org/@launchpad/goerli) (harder).

## Generate some keys

You’ll need four accounts and their private keys when setting up the chain:

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

You should get an output like the following: