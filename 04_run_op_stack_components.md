## Start op-node

Once you've got op-geth running you'll need to run op-node. Like Ethereum, the OP Stack has a Consensus Client (op-node) and an Execution Client (op-geth). The Consensus Client "drives" the Execution Client over the Engine API.

#### 1. Open up a new terminal

You'll need a terminal window to run the op-node in. Navigate to the op-node directory

```bash
cd ~/optimism/op-node
```

#### 2. Run op-node
```bash
    ./bin/op-node
    --l2=http://localhost:8551
    --l2.jwt-secret=./jwt.txt --sequencer.enabled
    --sequencer.l1-confs=5
    --verifier.l1-confs=4
    --rollup.config=./rollup.json  --rpc.addr=0.0.0.0 
    --rpc.port=8547 
    --p2p.disable  
    --rpc.enable-admin  
    --p2p.sequencer.key=$GS_SEQUENCER_PRIVATE_KEY 
    --l1=$L1_RPC_URL  
    --l1.rpckind=$L1_RPC_KIND
    --l1.trustrpc=true
```

Once you run this command, you should start seeing the op-node begin to sync L2 blocks from the L1 chain. Once the op-node has caught up to the tip of the L1 chain, it'll begin to send blocks to op-geth for execution. At that point, you'll start to see blocks being created inside of op-geth


## Start op-batcher

The op-batcher takes transactions from the Sequencer and publishes those transactions to L1. Once these Sequencer transactions are included in a finalized L1 block, they're officially part of the canonical chain. The op-batcher is critical!

It's best to give the Batcher address at least 1 ETH in native to ensure that it can continue operating without running out of ETH for gas. Keep an eye on the balance of the Batcher address because it can expend ETH quickly if there are a lot of transactions to publish.

#### 1. Open up a new terminal

You'll need a terminal window to run the op-batcher in. Navigate to the op-batcher directory 

```bash
cd ~/optimism/op-batcher
```

#### 2. Run op-batcher
```bash
./bin/op-batcher \
  --l2-eth-rpc=http://localhost:8545 \
  --rollup-rpc=http://localhost:8547 \
  --poll-interval=1s \
  --sub-safety-margin=6 \
  --num-confirmations=1 \
  --safe-abort-nonce-too-low-count=3 \
  --resubmission-timeout=30s \
  --rpc.addr=0.0.0.0 \
  --rpc.port=8548 \
  --rpc.enable-admin \
  --max-channel-duration=4 \
  --l1-eth-rpc=$L1_RPC_URL \
  --private-key=$GS_BATCHER_PRIVATE_KEY
```

_The `--max-channel-duration=n` setting tells the batcher to write all the data to L1 every `n` L1 blocks. When it is low, transactions are written to L1 frequently and other nodes can synchronize from L1 quickly. When it is high, transactions are written to L1 less frequently and the batcher spends less ETH. If you want to reduce costs, either set this value to 0 to disable it or increase it to a higher value._


## Start op-proposer

Now start op-proposer, which proposes new state roots.

#### 1. Open up a new terminal

You'll need a terminal window to run the op-proposer in. Navigate to the op-proposer directory

```bash
cd ~/optimism/op-proposer
```

#### 2. Run op-proposer
```bash
./bin/op-proposer \  --poll-interval=12s --rpc.port=8560  --rollup-rpc=http://localhost:8547  --l2oo-address=<L2OutputOracleProxyAddress>   --private-key=$GS_PROPOSER_PRIVATE_KEY  --l1-eth-rpc=$L1_RPC_URL
```

you'll be able to get L2OutputOracleProxyAddress from the path `/packages/contracts-bedrock/deployments/<l1-chain-id>-deploy.json`

In our the L1 chainId is `17000` which belongs to Holesky