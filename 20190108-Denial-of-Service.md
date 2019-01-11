## What happened
At block height 5371, the chain was stuck for 8 hours due to a Denial-of-Service that prevented all miners from mining any blocks.

The attack consisted of the following:

1. Broadcast an invalid block onto the network, which gets added to every miner's copy of the chain and verified later due to the expensive verification step.
  * Nodes stop mining before adding the block, and start again on the new block.
2. Broadcast a valid `NOTIFY_INVALID_CHECKPOINT` message
  * Nodes stop mining when such a message is broadcast and check the added block for this invalid hash checkpoint.
  * If the block was indeed invalid (as it is in this scenario), the chain is rolled back and mining restarts from the older block.
3. Rinse and repeat

Due to nodes not keeping a copy of invalid block IDs, they never realize they're adding and removing the same block over and over again, stopping all mining from taking place.

## Solutions
A short term solution is a list of invalid block IDs on each node, but due to the ease of creating new _invalid_ blocks, that would not solve it. Hence commit [527bd79](https://github.com/unprll-project/unprll/commit/527bd792da35c1ca9d644ab1bed25d42dd178a1f) implemented a measure by which block broadcasts require a signature from the miner's private spend key. Invalid hash checkpoints now block that miner's public key (`miner_specific`) in newer block broadcasts, hence preventing them from having any further blocks in the same wallet accepted for 24 hours.
