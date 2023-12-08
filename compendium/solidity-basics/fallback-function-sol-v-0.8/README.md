# Fallback function (sol v 0.8)

### Fallback()

* The `fallback()` function is a special function in Solidity that is used to define a default behavior if no other functions have been called.&#x20;
* It is triggered when a contract receives a transaction or message call and no other functions match the given _**signature**_.

It's mainly used to enable the contract to receive ETH.

```solidity
// This fallback function will keep all the Ether

fallback() external payable {
    balance[msg.sender] += msg.value;
} 
```

* must be marked `external payable`
* cannot take any parameters

### Receive v Fallback v payable

* To handle these scenarios when someone sends ETH directly to a contract, and does not call payable functions, we use receive and fallback functions.
* If someone sends ETH to a smart contract _**without passing calldata**_ and there is a `receive` function, the `receive` function is executed.&#x20;
* If there is no `receive` function, then the `fallback` is executed.&#x20;
* If there is no `receive` or `fallback`, then the transaction reverts.

<figure><img src="../../../.gitbook/assets/image (168).png" alt=""><figcaption></figcaption></figure>

* `receive` is for receiving ether without calling data.&#x20;

```solidity
// msg.data is empty
address.call{value : 1 ether}("");   
```

* payable function is for receiving ether with data&#x20;

```solidity
address.call{value : 1 ether}(abi.encodeWithSignature('fundme()');
```

{% hint style="danger" %}
If you want your contract to receive Ether, you should implement `receive.`

Using `fallback` for receiving Ether is not recommended, since it would catchall, and not fail on interface confusions.
{% endhint %}

