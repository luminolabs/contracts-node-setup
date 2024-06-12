## Configure your network

Once you’ve built both repositories, you’ll need head back to the Optimism Monorepo to set up the configuration for your chain. Currently, chain configuration lives inside of the [`contracts-bedrock`](https://github.com/ethereum-optimism/optimism/tree/129032f15b76b0d2a940443a39433de931a97a44/packages/contracts-bedrock) package.

1. Enter the Optimism Monorepo:

   ```bash
   cd ~/optimism
   ```

1. Move into the `contracts-bedrock` package:

   ```bash
   cd packages/contracts-bedrock
   ```

1. Inside of `optimism` repo, copy the environment file into the `contracts-bedrock` path

   ```sh
   cp .envrc.example .envrc
   ```

1. Fill out the environment variables inside of that file:

    - `ETH_RPC_URL` — URL for your L1 node.
    - `PRIVATE_KEY` — Private key of the `Admin` account.
    - `DEPLOYMENT_CONTEXT` - Name of the network, should be "getting-started"

1. Pull the environment variables into context using `direnv`

    ```bash
    direnv allow .
    ```

    If you need to install `direnv`, [make sure you also modify the shell configuration](https://direnv.net/docs/hook.html).

1. Before we can create our configuration file, we’ll need to pick an L1 block to serve as the starting point for our Rollup. It’s best to use a finalized L1 block as our starting block. You can use the `cast` command provided by Foundry to grab all of the necessary information:

    ```bash
    cast block finalized --rpc-url $ETH_RPC_URL | grep -E "(timestamp|hash|number)"
    ```

    You’ll get back something that looks like the following:

    ```
    hash                 0xd3091ed8a750029c1071693b8974ec1156bf5685d92adf5620ed857821869e02
    number               1717366
    timestamp            1718142912
    ```

1. Fill out the remainder of the pre-populated config file found at [`deploy-config/getting-started.json`](https://github.com/ethereum-optimism/optimism/blob/129032f15b76b0d2a940443a39433de931a97a44/packages/contracts-bedrock/deploy-config/getting-started.json). Use the default values in the config file and make following modifications:

    - Populate variable with their respective addresses and private keys that we generated earlier.
    - Replace `BLOCKHASH` with the blockhash you got from the `cast` command.
    - Replace `TIMESTAMP` with the timestamp you got from the `cast` command. Note that although all the other fields are strings, this field is a number! Don’t include the quotation marks.

## Deploy the L1 contracts

Once you’ve configured your network, it’s time to deploy the L1 smart contracts necessary for the functionality of the chain.

1. Create a `getting-started` deployment directory.

   ```bash
   mkdir deployments/getting-started
   ```


1. Once you’re ready, deploy the L1 smart contracts.

    ```bash
    forge script scripts/Deploy.s.sol:Deploy --private-key $PRIVATE_KEY --broadcast --rpc-url $ETH_RPC_URL
    forge script scripts/Deploy.s.sol:Deploy --sig 'sync()' --private-key $PRIVATE_KEY --broadcast --rpc-url $ETH_RPC_URL
    ```

Contract deployment can take up to 15 minutes. Please wait for all smart contracts to be fully deployed before continuing to the next step.

## Generate the L2 config files

We’ve set up the L1 side of things, but now we need to set up the L2 side of things. We do this by generating three important files, a `genesis.json` file, a `rollup.json` configuration file, and a `jwt.txt` [JSON Web Token](https://jwt.io/introduction) that allows the `op-node` and `op-geth` to communicate securely.

1. Head over to the `op-node` package.

    ```bash
    cd ~/optimism/op-node
    ```

1. Run the following command, and make sure to replace `<RPC>` with your L1 RPC URL:

    ```bash
    go run cmd/main.go genesis l2 \
        --deploy-config ../packages/contracts-bedrock/deploy-config/getting-started.json \
        --deployment-dir ../packages/contracts-bedrock/deployments/getting-started/ \
        --outfile.l2 genesis.json \
        --outfile.rollup rollup.json \
        --l1-rpc <RPC>
    ```

    You should then see the `genesis.json` and `rollup.json` files inside the `op-node` package.

 1. Next, generate the `jwt.txt` file with the following command:

    ```bash
    openssl rand -hex 32 > jwt.txt
    ```

1. Finally, we’ll need to copy the `genesis.json` file and `jwt.txt` file into `op-geth` so we can use it to initialize and run `op-geth` in just a minute:

    ```bash
    cp genesis.json ~/op-geth
    cp jwt.txt ~/op-geth
    ```
