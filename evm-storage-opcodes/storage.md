# Storage

Storage is setup based on 32 byte words, variables that are smaller than 32 bytes will be stored in the same word.

<figure><img src="../.gitbook/assets/image (221).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Other links:

* [https://noxx.substack.com/p/evm-deep-dives-the-path-to-shadowy-3ea?s=r](https://noxx.substack.com/p/evm-deep-dives-the-path-to-shadowy-3ea?s=r)
* [https://programtheblockchain.com/posts/2018/03/09/understanding-ethereum-smart-contract-storage/](https://programtheblockchain.com/posts/2018/03/09/understanding-ethereum-smart-contract-storage/)
{% endhint %}

### Layout of State Variables in Storage

* State variables of contracts are stored in storage in a compact way such that multiple values sometimes use the same storage slot.
* Data is stored contiguously, item after item, starting with the first state variable, which is stored in slot `0` (except for dynamically-sized arrays and mappings)
* Multiple, **contiguous items that need less than 32 bytes are packed into a single storage slot** if possible, according to the following rules:
  * The first item in a storage slot is stored lower-order aligned.
  * Value types use only as many bytes as are necessary to store them.
  * If a value type does not fit the remaining part of a storage slot, it is stored in the next storage slot.
  * **Structs and array data always start a new slot** and their items are packed tightly according to these rules.
  * Items following struct or array data always start a new storage slot.

<details>

<summary><mark style="color:red;">Beneficial to use reduced-size type (uint64), when dealing with storage values because the compiler will pack multiple elements into a single storage slot.</mark></summary>

* Thus **combine multiple reads or writes into a single operation → save gas**
* However, this is only sensible if all the variables in that slot are going to be used as a group (read/write) operation. (i.e. debt and deposit)
* **If you are not reading or writing all the values in a slot at the same time**, this can have the opposite effect
  * When one value is written to a multi-value storage slot, the storage slot has to be read first and then combined with the new value such that other data in the same slot is not destroyed.

</details>

{% hint style="info" %}
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time.

Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
{% endhint %}

To allow the EVM to optimize for this, ensure that you order storage variables and `struct` members such that they can be packed tightly.

For example, declaring your storage variables in the order of `uint128, uint128, uint256` instead of `uint128, uint256, uint128`, as the former will only take up two slots of storage whereas the latter will take up three.

### Inheritance

* For contracts that use inheritance, the ordering of state variables is determined by the C3-linearized order of contracts starting with the most base-ward contract.
* If allowed by the above rules, state variables from different contracts do share the same storage slot.
* [https://docs.soliditylang.org/en/latest/internals/layout\_in\_storage.html](https://docs.soliditylang.org/en/latest/internals/layout\_in\_storage.html)

<figure><img src="../.gitbook/assets/image (189).png" alt=""><figcaption></figcaption></figure>

