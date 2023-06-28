# gasLimit & min cost

<figure><img src="../.gitbook/assets/image (220).png" alt=""><figcaption></figcaption></figure>

**gasLimit**

&#x20;amount of gas that was implicitly purchases from the sender's account balance. basically, how much gas that has to be set aside for the transaction.

* txn is considered invalid if balance cannot support such a purchase
* any unused gas at the end of the txn is refunded

**Why?**

The nodes cannot look a txn and know how much gas it will require beforehand.&#x20;

They have to compute it to know much it will require. So it makes more sense for the customer to set a reasonably high gas limit that would cover the gas costs for a txn.&#x20;

### min gas cost: 21,000

The execution of a transaction defines a state transition. To that end, any transactions executed must first pass the initial tests of intrinsic validity, as mentioned in the yellow paper. These tests require 21,000 gas.

<figure><img src="../.gitbook/assets/image (55).png" alt=""><figcaption><p>requires 21,000 gas</p></figcaption></figure>

{% hint style="info" %}
Therefore, every transaction costs at least 21,000 gas
{% endhint %}
