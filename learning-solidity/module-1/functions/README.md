# Functions

## **Function Structure**

![Example](<../../../.gitbook/assets/image (176).png>)

* `public` -> visibility declaration&#x20;
* `view` -> Indicates function behavior
* `returns` -> Data type that the function returns (optional)

```solidity
function functionName(parameter1, parameter2) {public|private|internal|external} 
[pure|view|payable] [(modifiers)] [returns (<return types>)]
```

## Function Visibility

#### A function declarations only impose scope and do nothing for security

![](<../../../.gitbook/assets/image (72).png>)

### Simply put:

**public ->** functions can be called both internally and externally&#x20;

**external ->** only external calls; cannot be accessed internally

**internal ->** only this contract and contracts deriving from it can access

**private ->** functions can only be called from within the contract

{% hint style="danger" %}
Cannot create a function with the same name as a storage variable. Will conflict with getter function.
{% endhint %}
