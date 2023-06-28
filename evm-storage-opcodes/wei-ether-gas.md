# Wei, Ether, Gas

## Gas v Gas Price

* 1 Eth = 10\*\*18 Wei&#x20;
* 1 Eth = 10\*\*9 gwei (giga-wei)

<figure><img src="../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

**Each operation in a contract costs gas: Gas Costs from Yellow Paper**

{% hint style="info" %}
💡 \
ADD .... 3

MUL .... 5

SUB .... 3

DIV .... 5
{% endhint %}

So to do a+b, I would need 3 units of gas. But this doesn't tell me how much eth I would pay. That would be determined by **gasPrice**.

## **gasPrice**

**The gasPrice is the value that the transaction sender is willing to pay per unit of gas.**

* Transaction sender is capable of choosing how much he wants to pay per unit of gas.
* If our transaction needs 3 gas and we are willing to pay 10 Wei per unit of gas, our transaction cost would be 30 Wei in total.
* unit = wei/gas

{% hint style="info" %}
💡 The real-world costs of 30 Wei would be determined in the open market -> ETH/USDT
{% endhint %}

**startGas/gasLimit**

This is the total units of gas we would want to spend on a specific transaction. Our max total spend.

**Why do we need to specify this?**

It is not easy to calculate gas costs for certain functions/operations ahead of time. For example, a for loop that runs over a collection of items that could grow or shrink over time.

**Example**

<figure><img src="../.gitbook/assets/image (172).png" alt=""><figcaption></figcaption></figure>

* We need to spend 14 gas, and are willing to spend 20 gas. Transaction will be successfully processed.
* Since we decided to pay 300 wei/gas (gasPrice), our total cost comes to 300 \* 14 = 4,200 wei

{% hint style="info" %}
💡 If we had opted for gasLimit = 10, the function would have halted after the second step, as that is as far as it could go. \<Transaction Failed>
{% endhint %}

## **Mining**

In Ethereum, when a miner mines a new block, it receives the fees from all transactions included in this block. Therefore, **the higher the gasPrice in the transactions, the higher the fees that the miner receives will be.** The miner also receives a fixed reward per block and a reward for including uncles in the block.

Valid uncle blocks are rewarded **to neutralize the effect of network lag on the distribution of mining rewards**.[https://github.com/ethereum/wiki/wiki/Mining#mining-rewards](https://github.com/ethereum/wiki/wiki/Mining#mining-rewards)

Bob creates the transaction with gasLimit = 100 and gasPrice = 2. Unfortunately, John only has 100 Wei, he can’t set the gasLimit to 200 because that would make the transaction intrinsic cost higher than his current balance. John creates the transaction with gasLimit = 100 and gasPrice = 1.

When it is time to pick a transaction to include in the next block, the miner node is likely to choose the transaction that will reward him more fees. In our example, Bob has set a gasPrice twice as high as John’s gasPrice. Since both transactions have the same gas cost, the miner will receive twice as much Wei as a reward if it chooses Bob’s transaction.

<figure><img src="../.gitbook/assets/image (165).png" alt=""><figcaption></figcaption></figure>

This mechanism of charging the transaction sender and rewarding the miner creates a self-regulated economy. The senders are always trying to minimise fees and the miners always trying to maximize their reward. When sending a transaction, you can set a higher gasPrice to make mining this transaction more interesting to miners, resulting in the transaction being mined faster.

Some miners even have a minimum gasPrice, meaning they ignore any transactions with a gasPrice lower than what they want.

When sending a transaction, it can be hard to know what is the minimum gasPrice at that moment. [There are some tools](https://ethgasstation.info/) that scan the network and the average gasPrice used in recent transactions to help with choosing a fair gasPrice that is likely to be accepted by miners.

**Describe the difference between gas cost and gas price.**

* Gas is the number of units of computational effort required to execute specific operations on Ethereum
* Gas price is the cost of each unit of gas

**What is the max gas of each block?**

* Each block has a target size of 15 million gas, but the size of blocks will increase or decrease according to network demand, up until the block limit of 30 million gas

**What happens when the gas limit specified is more than the gas consumed?**

* Gas limit refers to the maximum amount of gas you are willing to consume on a transaction. Any gas not used in a transaction is refunded to the user.

**What happens to gas when a transaction fails / if there is not enough gas in a transaction?**

* If too little gas is specified, the EVM will consume all the gas units attempting to fulfill the transaction, but it will not complete. The EVM then **reverts any changes**, but since the validator has already done 20k gas units worth of work, that gas is consumed.

**What are some methods to optimize gas?**

* Compared to regular state variables, the gas costs of constant and immutable variables are much lower. For a constant variable, the expression assigned to it is copied to all the places where it is accessed and also re-evaluated each time. This allows for local optimizations. Immutable variables are evaluated once at construction time and their value is copied to all the places in the code where they are accessed.
