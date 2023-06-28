# Global variables

* `msg.sender` — The _msg.sender_ is the address that has called or initiated a function or created a transaction
* `msg.data` — The complete `calldata` which is a non-modifiable, non-persistent area where function arguments are stored and behave mostly like `memory`
* `msg.gas` — Returns the available gas remaining for a current transaction (you can learn more about gas in Ethereum [here](https://www.cryptocompare.com/coins/guides/what-is-the-gas-in-ethereum/))
* `msg.sig` — The first four bytes of the calldata for a function that specifies the function to be called (i.e., function selector)
* `msg.value` — The amount of wei sent with a message to a contract (wei is a denomination of ETH)

<figure><img src="../../.gitbook/assets/image (285).png" alt=""><figcaption></figcaption></figure>

### msg.sender vs tx.origin

There are two types of accounts in the Ethereum Chain

1. Externally Owned Accounts (EOA) \[Person]
2. Contracts Accounts \[Contracts on Chain]

Both accounts have addresses.

<figure><img src="../../.gitbook/assets/image (287).png" alt=""><figcaption></figcaption></figure>

\


\
