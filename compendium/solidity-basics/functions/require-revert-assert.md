# Require, Revert, Assert

Require is a keyword in Solidity that is used to make sure that functions are called with the correct arguments.&#x20;

* `require` declares constraints which must be satisfied before code execution.
* It accepts a single argument and **returns a boolean** value after evaluation.&#x20;
* It also has a custom error string option.

```solidity
contract ExampleContract {    
    function exampleFunction(uint a, uint b) public {        
        require(a > b);    
    }
}
```

If the condition evaluates to false then exception is raised and execution is terminated. The unused gas is returned back to the caller and the state is reversed to its original state.

### Use `require()` to:

* Validate function inputs
* Validate the response from an external contract
* Validate state conditions prior to executing state changing operations

### Require and Revert

* Same thing, user choice.
* `revert("some error string")` aborts execution and reverts state changes, providing an explanatory string.

```solidity
require(amount <= msg.value, "Not enough ETH");

//or

if (amount < msg.value){
    revert("Not enough ETH");
}

// require just seems more tidy
```

* there has been a shift to using custom errors paired with revert due to gas savings.

## Transactions and Errors

* Errors are state-reverting
* If an error is encountered everything that was done before is undone.
* Entire transaction fails -> no state change is effected

### Revert cascades

* Say your smart contract sends a transaction to another smart contract.&#x20;
* An exception is thrown in the external contract.&#x20;
* As a result, all actions taken in both the external contract and your contract are reverted.

The error cascades as the entire flow of actions is treated as a single transaction. This applies to all high-level interactions/functions.&#x20;

### Low-level calls do not revert

* Low-level calls: Address.`send`, address.`call`, address.`delegatecall`, address.`staticcall`

#### Example

Let’s say that you have a contract A which makes a low-level `call` or `delegatecall` to another contract B. The target contract B reverts with a revert message or custom error.

{% tabs %}
{% tab title="contract A" %}
```solidity
contract A {
  function foo(address target, bytes data) external {
      // used to call B.bar()
      (bool success, bytes memory result) = target.delegatecall(data) 
  }
}
```
{% endtab %}

{% tab title="contract B" %}
```solidity
contract B {
  error AccessForbidden(address sender);

  function bar() external {
    revert AccessForbidden(msg.sender);
  }
}
```
{% endtab %}
{% endtabs %}

It’s important to know that a low-level `call` or `delegatecall` doesn’t revert while calling a function that reverts:

```solidity
// These won't revert even if the target contract reverts!
(bool success, bytes memory result) = target.call(data);
(bool success, bytes memory result) = target.delegatecall(data);
```

In the above example, when contract A calls contract B with a low-level call, The `success` variable signals whether the call was successful (`true`) or unsuccessful (`false`).&#x20;

Using this, we could revert in the calling contract like so:

```solidity
contract A {
  function foo(address target, bytes data) external {
    (bool success, bytes memory result) = target.delegatecall(data);
    require(success);
  }
}
```

## Use `assert()` to:

* check for overflow/underflow
* check invariants  (states our contract or variables should never ever reach)
* validate contract state _after_ making changes
* avoid conditions which should never, ever be possible.
* Generally, you should use `assert` less often

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Error {
    function testRequire(uint _i) public pure {
        // Require should be used to validate conditions such as:
        // - inputs
        // - conditions before execution
        // - return values from calls to other functions
        require(_i > 10, "Input must be greater than 10");
    }

    function testRevert(uint _i) public pure {
        // Revert is useful when the condition to check is complex.
        // This code does the exact same thing as the example above
        if (_i <= 10) {
            revert("Input must be greater than 10");
        }
    }

    uint public num;

    function testAssert() public view {
        // Assert should only be used to test for internal errors,
        // and to check invariants.

        // Here we assert that num is always equal to 0
        // since it is impossible to update the value of num
        assert(num == 0);
    }

    // custom error
    error InsufficientBalance(uint balance, uint withdrawAmount);

    function testCustomError(uint _withdrawAmount) public view {
        uint bal = address(this).balance;
        if (bal < _withdrawAmount) {
            revert InsufficientBalance({balance: bal, withdrawAmount: _withdrawAmount});
        }
    }
}
```

* [https://ethereum-blockchain-developer.com/027-exceptions/04-invariants-with-assert/](https://ethereum-blockchain-developer.com/027-exceptions/04-invariants-with-assert/)&#x20;

### Require and Assert

Both terminate and revert the transaction if some condition is not met. However, they have different compiled EVM bytecode:

* `require(false)` compiles to `0xfd` which is the `REVERT` opcode, meaning it will **refund** the remaining gas and revert all changes.&#x20;
  * The opcode can also return a value (useful for debugging).
* `assert(false)` compiles to `0xfe`, which is an invalid opcode, **consuming** all remaining gas, and reverting all changes.
