## Configure your network

Once you’ve built both repositories, you’ll need head back to the Optimism Monorepo to set up the configuration for your chain. Currently, chain configuration lives inside of the [`contracts-bedrock`](https://github.com/ethereum-optimism/optimism/tree/129032f15b76b0d2a940443a39433de931a97a44/packages/contracts-bedrock) package.

1. Enter the Optimism Monorepo:

   ```bash
   cd ~/optimism
   ```

2. Inside of `optimism` repo, copy the environment file into the `.envrc` path

   ```sh
   cp .envrc.example .envrc
   ```

_Note: If you think there any env variable missing just add them inside the .envrc file_

3. Fill out the environment variables inside of that file:

    - `L1_RPC_URL` or `ETH_RPC_URL` — URL for your L1 node.
    - `PRIVATE_KEY` — Private key of the `Admin` account.
    - `DEPLOYMENT_CONTEXT` - Name of the network, should be anything like "lumino-test"

4. Pull the environment variables into context using `direnv`

    ```bash
    direnv allow .
    ```

If you need to install `direnv`, [make sure you also modify the shell configuration](https://direnv.net/docs/hook.html).

5. Before we can create our configuration file, we’ll need to pick an L1 block to serve as the starting point for our Rollup. It’s best to use a finalized L1 block as our starting block. You can use the `cast` command provided by Foundry to grab all of the necessary information:

    ```bash
    cast block finalized --rpc-url $ETH_RPC_URL | grep -E "(timestamp|hash|number)"
    ```

    You’ll get back something that looks like the following:

    ```
    hash                 0xd3091ed8a750029c1071693b8974ec1156bf5685d92adf5620ed857821869e02
    number               1717366
    timestamp            1718142912
    ```

1. Fill out the remainder of the `pre-populated` .envrc variable. Use the default values in the config file and make following modifications:

- Populate variable with their respective addresses and private keys that we generated earlier.
- Replace `BLOCKHASH` with the blockhash you got from the `cast` command.
- Replace `TIMESTAMP` with the timestamp you got from the `cast` command. Note that although all the other fields are strings, this field is a number! Don’t include the quotation marks.

## Generate the configuration file

Once you've built both repositories, you'll need to head back to the Optimism Monorepo to set up the configuration file for your chain. Currently, chain configuration lives inside of the [contracts-bedrock](https://github.com/ethereum-optimism/optimism/tree/v1.1.4/packages/contracts-bedrock) package in the form of a JSON file.

1. Enter the Optimism Monorepo

```bash
cd ~/optimism
```

2. Move into the contracts-bedrock package

```bash
cd packages/contracts-bedrock
```

3. Generate the configuration file
Run the following script to generate the getting-started.json configuration file inside of the deploy-config directory.

_Note: change the name of the output at the bottom/last line of the script if you want a custom named config file_

```bash
./scripts/getting-started/config.sh
```

After running the script correctly we should see a file that is named something like <custom-name>.json

4. Add this line to your .envrc

```bash
export DEPLOY_CONFIG_PATH=./deploy-config/<custom-name>.json
```

## Deploy the Create2 Factory (Optional)

If you're deploying an OP Stack chain to a network other than Sepolia/Holesky, you may need to deploy a Create2 factory contract to the L1 chain. This factory contract is used to deploy OP Stack smart contracts in a deterministic fashion.

_This step is typically only necessary if you are deploying your OP Stack chain to custom L1 chain. If you are deploying your OP Stack chain to Sepolia, you can safely skip this step._

#### 1. Check if the factory exists
The Create2 factory contract will be deployed at the address `0x4e59b44847b379578588920cA78FbF26c0B4956C`. You can check if this contract has already been deployed to your L1 network with a block explorer or by running the following command:

```bash
cast codesize 0x4e59b44847b379578588920cA78FbF26c0B4956C --rpc-url $L1_RPC_URL
```

If the command returns `0` then the contract has not been deployed yet. If the command returns `69` then the contract has been deployed and you can safely skip this section.

#### 2. Fund the factory deployer
You will need to send some ETH to the address that will be used to deploy the factory contract, `0x3fAB184622Dc19b6109349B94811493BF2a45362`. This address can only be used to deploy the factory contract and will not be used for anything else. Send at least 1 ETH to this address on your L1 chain.

#### 3. Deploy the factory
Using cast, deploy the factory contract to your L1 chain:

```bash
cast publish --rpc-url $L1_RPC_URL 0xf8a58085174876e800830186a08080b853604580600e600039806000f350fe7fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe03601600081602082378035828234f58015156039578182fd5b8082525050506014600cf31ba02222222222222222222222222222222222222222222222222222222222222222a02222222222222222222222222222222222222222222222222222222222222222
``` 

Wait for the transaction to be mined
Make sure that the transaction is included in a block on your L1 chain before continuing.

#### 4. Verify that the factory was deployed

Run the code size check again to make sure that the factory was properly deployed:

```bash
cast codesize 0x4e59b44847b379578588920cA78FbF26c0B4956C --rpc-url $L1_RPC_URL
```


## Deploy the L1 contracts

Once you’ve configured your network, it’s time to deploy the L1 smart contracts necessary for the functionality of the chain.

1. Create a `getting-started` deployment directory.

   ```bash
   mkdir deployments/lumino-test
   ```


2. Once you’re ready, deploy the L1 smart contracts.

    ```bash
    forge script scripts/Deploy.s.sol:Deploy --private-key $GS_ADMIN_PRIVATE_KEY --broadcast --rpc-url $L1_RPC_URL --slow
    ```

_If you see a nondescript error that includes `EvmError: Revert` and `Script failed` then you likely need to change the `IMPL_SALT` environment variable. This variable determines the addresses of various smart contracts that are deployed via [CREATE2](https://eips.ethereum.org/EIPS/eip-1014). If the same `IMPL_SALT` is used to deploy the same contracts twice, the second deployment will fail. You can generate a new `IMPL_SALT` by running direnv allow anywhere in the Optimism Monorepo._

<!-- 3. Generate contract artifacts:
    
    ```bash
    forge script scripts/Deploy.s.sol:Deploy --sig 'sync()' --rpc-url $L1_RPC_URL
    ``` -->

Contract deployment can take up to 15-30 minutes. Please wait for all smart contracts to be fully deployed before continuing to the next step.

## Generate the L2 config files

We’ve set up the L1 side of things, but now we need to set up the L2 side of things. We do this by generating three important files, a `genesis.json` file, a `rollup.json` configuration file, and a `jwt.txt` [JSON Web Token](https://jwt.io/introduction) that allows the `op-node` and `op-geth` to communicate securely.

1. Generate `state-dump` and `l2-allocs`

```bash
cd ~/optimism/packages/contracts-bedrock

CONTRACT_ADDRESSES_PATH=./deployments/17000-deploy.json DEPLOY_CONFIG_PATH=./deploy-config/lumino-test.json STATE_DUMP_PATH=./deployments/l2-allocs.json  forge script scripts/L2Genesis.s.sol:L2Genesis  --sig 'runWithStateDump()'
```

2. Head over to the `op-node` package.

    ```bash
    cd ~/optimism/op-node
    ```

1. Run the following command, and make sure to replace `<RPC>` with your L1 RPC URL:

    ```bash
    go run cmd/main.go genesis l2 
    --l1-rpc <l1-rpc-value>
    --deploy-config ../packages/contracts-bedrock/deploy-config/lumino-test.json 
    --l2-allocs ../packages/contracts-bedrock/deployments/l2-allocs.json 
    --l1-deployments ../packages/contracts-bedrock/deployments/17000-deploy.json 
    --outfile.l2 genesis.json  
    --outfile.rollup rollup.json
    ```

    You should then see the `genesis.json` and `rollup.json` files inside the `op-node` package.

 4. Next, generate the `jwt.txt` file with the following command:

    ```bash
    openssl rand -hex 32 > jwt.txt
    ```

5. Finally, we’ll need to copy the `genesis.json` file and `jwt.txt` file into `op-geth` so we can use it to initialize and run `op-geth` in just a minute:

    ```bash
    cp genesis.json ~/op-geth
    cp jwt.txt ~/op-geth
    ```
