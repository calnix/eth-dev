---
description: for variables
---

# Control Variable Visibility

Visibility modifiers **restrict** who can use values of Solidity variables. Here is a list of modifiers that change these permissions:

* `public`: anyone can get the value of a variable.
* `external`: only external functions can get the value of a local variable. It is not used on state variables.
* `internal`: only functions in this contract and inherited contracts can get values.
* `private`: access limited to functions from this contract.

{% hint style="warning" %}
**public = external  + internal**

public functions cost more gas than either external or internal. To save gas, restrict to external/internal when possible.
{% endhint %}

### public vs external

`public` and `external` differs in terms of gas usage. The former use more than the latter when used with large arrays of data. This is due to the fact that Solidity copies arguments to memory on a `public` function while `external` read from `calldata` which is cheaper than memory allocation.

* `public` functions are called **internally and externally**
* &#x20;`internal` calls are executed via jumps in the code because array arguments are passed internally by pointers to memory.&#x20;
* When the compiler generates the code for an internal function, that function expects its arguments to be located in memory.&#x20;
* That is why `public` functions are allocated to memory.&#x20;
* **The optimization that happens with `external` is that is does not care for the internal calls.**

{% hint style="info" %}
So if you know the function you create only allows for `external` calls, go for it. It provides performance benefits and you will save on gas.
{% endhint %}

## Immutable v Constant

* for constant variables, the value has to be fixed at compile-time, (initial and assign in one line)
* for immutable, it can still be assigned at construction time (within constructor)

```solidity
IERC20 public immutable collateral;    
IERC20 public immutable debt;  

constructor(address dai_, address weth_, address priceFeedAddress, uint bufferNumerator_, uint bufferDenominator_) {
    collateral = IERC20(weth_);
    debt = IERC20(dai_);
    priceFeed = AggregatorV3Interface(priceFeedAddress);
    // collateralization level
    bufferNumerator = bufferNumerator_;
    bufferDenominator = bufferDenominator_;
}
```
