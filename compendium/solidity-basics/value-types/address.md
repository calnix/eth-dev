# address

address has 2 important properties:

* .balance
* .transfer(amount)
  * cascades exceptions
  * if an exception is triggered in the contract you are calling, typically this exception will cascade and the whole transaction will get cancelled. No state changes will be made.&#x20;

```solidity
address myAddress = "0xabc.."

myAddress.balance => balance in wei
myAddress.transfer(wei_amt) => transfer from the smart contract to the address
```

{% hint style="info" %}
for usage see function-payable
{% endhint %}

### Low-level calls

* `.send` -> returns a boolean, does not cascade exceptions.
  * does not cascade exceptions. if exception returns False.
* `.call.gas().value()()`
  * let's you forward a specific amount of gas.
  * returns boolean
  * you see this in re-entrancy bugs

#### .send & .transfer both only transfer 2300 gas along

* execution is capped with 2300 gas, to do whatever computation you want.
* this could cause issues if more than 2300 gas is required.&#x20;

