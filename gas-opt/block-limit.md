# Block Limit

**Bitcoin historically has limited the block size to 1MB**

* if the block becomes too big, it makes it difficult to synchronize across the world

**Ethereum does not explicitly set a byte limit to the block size. Instead it limits the amount of computations per block, or gas.**

* Each computation has a gas cost.
* All the computation within a block must fall within a certain gas threshold
* Else, it will be difficult for nodes to verify the transactions quickly.

{% hint style="info" %}
At time of writing, block limit is \~30 million gas. \
An eth transfer costs 21,000 gas, so theoretically, one block can hold 1,428 transfers.
{% endhint %}

* If your smart contract requires over 30 million gas to execute, it won't fit on the block.
* If there are more than 1428 people trying to make a transfer, then the highest bidder will have their transaction included on the block. This is why gas prices fluctuate.

{% hint style="info" %}
30 million gas is not a hard limit. The gas limit changes, and is not completely static.\
Ethereum technically has a preferred block limit and dynamic block limit but conceptually, you can think of it as 30 million gas.
{% endhint %}

### Minimum gas cost

* it costs 21,000 gas to do anything at the bare minimum: calling a fn, transfer, etc

### The cost of doing nothing

```solidity
contract DoNothing {

    // execution costs: 21138 gas
    function doNothing() external payable {}
}
```

**How do we get 21138 gas?**

* tx cost: 21,000
* opcodes: 65 gas
* tx.data 64 gas&#x20;
  * msg.data | tx.data submitted&#x20;
* memory: 9 gas &#x20;
  * free memory pointer: PUSH 80 PUSH 40 MSTORE
  * you are charged for memory expansion, even if you didn't write in the intermediate slots
  * we are using slots 0x040 to 0x60
  * 0x60 / 0x20 = 3 (96/32) in dec
  * 3 slots \* 3 gas = 9 gas

### Summary

There are 5 places to save gas

* **On deployment:** The smaller the contract, the less you pay on deployment
* **During computation:** Using fewer/cheaper op-codes saves gas on execution
* **Transaction data:** The larger your tx.data, the more non-zero bytes in it, the more gas costs.
* **Memory:** The more memory you alocate, even if you don't use it, you pay more gas.
* **Storage:** More storage used, more gas used.&#x20;

