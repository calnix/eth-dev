# Reading txn input data

Any transaction with data payload is meant to be addressed to a smart contract. These types of transactions usually invoke a function on smart contract.

![](<.gitbook/assets/image (368).png>)

The data payload is the hex data we see here. It is bytecode that the EVM will understand and execute. Let's breakdown the transaction data:

{% code overflow="wrap" %}
```bash
0xe2bbb15800000000000000000000000000000000000000000000000175150904d98ada050000000000000000000000000000000000000000000000000000000000000000

0x
e2bbb158
00000000000000000000000000000000000000000000000175150904d98ada05
050000000000000000000000000000000000000000000000000000000000000000
```
{% endcode %}

<figure><img src=".gitbook/assets/image (369).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
32 bytes == uint256

* 256 bits -> 64 hex characters    \[1 hex -> 4 bits]
* 1 byte -> 2 hex characters -> 8 bits
{% endhint %}
