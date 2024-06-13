## Initialize op-geth

We’re almost ready to run our chain! Now we just need to run a few commands to initialize `op-geth`. We’re going to be running a Sequencer node, so we’ll need to import the `Sequencer` private key that we generated earlier. This private key is what our Sequencer will use to sign new blocks.

1. Head over to the `op-geth` repository:

    ```bash
    cd ~/op-geth
    ```

2. Create a data directory folder:

    ```bash
    mkdir datadir
    ```

1. Next we need to initialize `op-geth` with the genesis file we generated and copied earlier:

    ```bash
    build/bin/geth init --datadir=datadir genesis.json
    ```

Everything is now initialized and ready to go!

## Run the node software

There are four components that need to run for a rollup.
The first two, `op-geth` and `op-node`, have to run on every node.
The other two, `op-batcher` and `op-proposer`, run only in one place, the sequencer that accepts transactions.

Set these environment variables for the configuration

| Variable       | Value |
| -------------- | -
| `SEQ_KEY`      | Private key of the `Sequencer` account
| `BATCHER_KEY`  | Private key of the `Batcher` accounts, which should have at least 1 ETH
| `PROPOSER_KEY` | Private key of the `Proposer` account
| `L1_RPC`       | URL for the L1 (such as Goerli) you're using
| `RPC_KIND`     | The type of L1 server to which you connect, which can optimize requests. Available options are `alchemy`, `quicknode`, `parity`, `nethermind`, `debug_geth`, `erigon`, `basic`, and `any`
| `L2OO_ADDR`    | The address of the `L2OutputOracleProxy`, available at `~/optimism/packages/contracts-bedrock/deployments/getting-started/L2OutputOracleProxy.json`

## Start `op-geth`

#### 1. Run `op-geth` with the following commands.

_You're using `--gcmode=archive` to run `op-geth` here because this node will act as your Sequencer. It's useful to run the Sequencer in archive mode because the `op-proposer` requires access to the full state. Feel free to run other (non-Sequencer) nodes in full mode if you'd like to save disk space._

It's important that you've already initialized the geth node at this point as per the previous section. Failure to do this will cause startup issues between `op-geth` and `op-node`.


```bash
cd ~/op-geth

./build/bin/geth \
  --datadir ./datadir \
  --http \
  --http.corsdomain="*" \
  --http.vhosts="*" \
  --http.addr=0.0.0.0 \
  --http.api=web3,debug,eth,txpool,net,engine \
  --ws \
  --ws.addr=0.0.0.0 \
  --ws.port=8546 \
  --ws.origins="*" \
  --ws.api=debug,eth,txpool,net,engine \
  --syncmode=full \
  --gcmode=archive \
  --nodiscover \
  --maxpeers=0 \
  --networkid=42069 \
  --authrpc.vhosts="*" \
  --authrpc.addr=0.0.0.0 \
  --authrpc.port=8551 \
  --authrpc.jwtsecret=./jwt.txt \
  --rollup.disabletxpoolgossip=true
```

And `op-geth` should be running! You should see some output, but you won’t see any blocks being created yet because `op-geth` is driven by the `op-node`. We’ll need to get that running next.