# Require

Require is a keyword in Solidity that is used to make sure that functions are called with the correct arguments.&#x20;

* `require` declares constraints which must be satisfied before code execution.
* It accepts a single argument and **returns a boolean** value after evaluation.&#x20;
* It also has a custom error string option.

```solidity
contract ExampleContract {    
    function exampleFunction(uint a, uint b) public {        
        require(a > b);    
     // require(a > b, "a is not larger than b");    
    }
}
```

* If the condition evaluates to false then exception is raised and execution is terminated.&#x20;
* The unused gas is returned back to the caller and the state is reversed to its original state.

{% hint style="warning" %}
Keep error strings to be under 32 characters for gas savings
{% endhint %}

### Use `require()` to:

* Validate function inputs
* Validate the response from an external contract
* Validate state conditions prior to executing state changing operations

### Require and Revert

* Same thing, user choice.
* `revert("some error string")` aborts execution and reverts state changes, providing an explanatory string.

```solidity
require(amount <= msg.value, "Not enough ETH");

// or:

if (amount < msg.value){
    revert("Not enough ETH");
}

// require just seems more tidy
```

* There has been a shift to using custom errors paired with revert due to gas savings.

