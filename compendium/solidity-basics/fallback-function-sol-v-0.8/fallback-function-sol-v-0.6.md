# Fallback function (sol v 0.6)

is executed if none of the other functions match the function identifier or no data was provided with the function call.&#x20;

Only one unnamed function can be assigned to a contract and it is executed whenever the contract receives plain Ether without any data.

```solidity
// This fallback function will keep all the Ether

function() external payable {
    balance[msg.sender] += msg.value;
} 
```

### Properties of a fallback function:

* Has no name or arguments.&#x20;
* If it is not marked payable, the contract will throw an exception if it receives plain ether without data.&#x20;
* Cannot return anything.&#x20;
* Can only be defined once per contract.&#x20;
* It is also executed if the caller meant to call a function that is not available.
* Mandatory to mark it external.&#x20;
* Limited to 2300 gas when called by another function - so as to make this function call as cheap as possible.

### Receiving ETH&#x20;

Contracts receiving ETH without a payable function call and without a fallback function will throw an exception. Therefore, cannot receive ETH.

However, there are some exceptions to this; You cannot completely avoid receiving ETH:

1. Some other SC selfdestructs and names you as beneficiary
2. Miner reward sets your SC address as beneficiary

{% hint style="info" %}
Prior Solidity 0.6 the fallback function was simply an anonymous function that looked like this:

```
function () external {}
```

It's now two different functions. **`receive()`** to receive money and **`fallback()`** to just interact with the Smart Contract without receiving Ether.&#x20;
{% endhint %}
