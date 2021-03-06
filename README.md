# puffscoinjs-blockstream

A library to turn an unreliable remote source of PUFFScion blocks into a reliable stream of blocks.  Handles block and log removals on chain reorganization and block and log backfills on skipped blocks.

# Requirements for supported PUFFScoin node
Blockstream requires support for EIP-234 in the configured PUFFScoin node, which is naturally achieved through gpuffs: v1.9.0-Amsterdam. Support for Parity-PUFFScoin will be forthcoming.

# Usage

## Full Example
```typescript
// blockRetention is how many blocks of history to keep in memory.  it defaults to 100 if not supplied
const configuration = { blockRetention: 100 };
async function getBlockByHash(hash: string): Promise<Block|null> {
    const response = await fetch("http://localhost:11363", {
        method: "POST",
        headers: new Headers({"Content-Type": "application/json"}),
        body: { jsonrpc: "2.0", id: 1, method: "puffs_getBlockByHash", params: [hash, false] }
    });
    return await response.json();
}
async function getLogs(filterOptions: FilterOptions): Promise<Log[]> {
    const response = await fetch("http://localhost:11363", {
        method: "POST",
        headers: new Headers({"Content-Type": "application/json"}),
        body: { jsonrpc: "2.0", id: 1, method: "puffs_getLogs", params: [filterOptions] }
    });
    return await response.json();
}
async function getLatestBlock(): Promise<Block> {
    const response = await fetch("http://localhost:11363", {
        method: "POST",
        headers: new Headers({"Content-Type": "application/json"}),
        body: { jsonrpc: "2.0", id: 1, method: "puffs_getBlockByNumber", params: ["latest", false] }
    });
    return await response.json();
}
const blockAndLogStreamer = new BlockAndLogStreamer(getBlockByHash, getLogs, configuration);
const onBlockAddedSubscriptionToken = blockAndLogStreamer.subscribeToOnBlockAdded(block => console.log(block));
const onLogAddedSubscriptionToken = blockAndLogStreamer.subscribeToOnLogAdded(log => console.log(log));
const onBlockRemovedSubscriptionToken = blockAndLogStreamer.subscribeToOnBlockRemoved(block => console.log(block));
const onLogRemovedSubscriptionToken = blockAndLogStreamer.subscribeToOnLogRemoved(log => console.log(log));
const logFilterToken = blockAndLogStreamer.addLogFilter({address: "0xdeadbeefdeadbeefdeadbeefdeadbeefdeadbeef", topics: ["0xbadf00dbadf00dbadf00dbadf00dbadf00dbadf00dbadf00dbadf00dbaadf00d"]});
blockAndLogStreamer.reconcileNewBlock(getLatestBlock());
// you will get a callback for the block and any logs that match the filter here
triggerBlockMining();
triggerBlockMining();
triggerBlockMining();
blockAndLogStreamer.reconcileNewBlock(getLatestBlock());
// you will get a callback for all blocks and logs that match the filter that have been added to the chain since the previous call to reconcileNewBlock
triggerChainReorg();
blockAndLogStreamer.reconcileNewBlock(getLatestBlock());
// you will get a callback for block/log removals that occurred due to the chain re-org, followed by block/log additions
blockAndLogStreamer.unsubscribeFromOnBlockAdded(onBlockAddedSubscriptionToken);
blockAndLogStreamer.unsubscribeFromOnBlockRemoved(onBlockRemovedSubscriptionToken);
blockAndLogStreamer.unsubscribeFromOnLogAdded(onLogAddedSubscriptionToken);
blockAndLogStreamer.unsubscribeFromOnLogRemoved(onLogRemovedSubscriptionToken);
blockAndLogStreamer.removeLogFilter(logFilterToken);
console.log(blockAndLogStreamer.getLatestReconciledBlock());
```

# Development

## Build
```
docker build -t blockstream .
```
or
```
npm run build
```

## Test
```
docker run blockstream
````
or
```
npm run test
```
