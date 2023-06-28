# call, staticcall, delegatecall

## Call

* `call` is a low level function to interact with other contracts.

> ```solidity
> bytes memory data = abi.encodeWithSignature(
>             "approve(address,uint256)", address(this), type(uint256).max
>         );
> (bool success, bytes memory data) = target.call{value: 111, gas: 5000}(data)
> ```

* value: amount of ETH to send to the target (in case you are calling a payable function).
* gas: optional value for gas to pass in to limit how much gas you could spend on the entire call

#### Example

```solidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

contract TestCall {
    string public message;
    uint public x;
    event Log(string message);

    fallback() external payable {
        emit Log("Fallback was called");
    }

    function foo(string memory _message, uint256 _x) public payable returns (bool, uint) {
        message = _message;
        x =  _x;
        return (true, 999);
    }
}

contract Call {
    bytes public data;
    
    function callFoo(address _test) external {
        (bool succes, bytes memory _data) = _test.call{value: 111, gas: 5000}(
            abi.encodedWithSignature("foo(string,uint256)", "call Foo", 123));
        require(success, "call failed");
        data = _data;
    }
}

```

* value: how much ether to send
* gas: how much gas to send
* &#x20;mark `callFoo` as payable so that we can send it ether when calling it&#x20;

{% hint style="info" %}
Read more:

* [https://ethereum.stackexchange.com/questions/136075/when-to-use-call-and-abi-encodewithsignature-when-calling-external-function-and](https://ethereum.stackexchange.com/questions/136075/when-to-use-call-and-abi-encodewithsignature-when-calling-external-function-and)
* [https://ethereum.stackexchange.com/questions/145345/all-about-call-usage](https://ethereum.stackexchange.com/questions/145345/all-about-call-usage)
{% endhint %}

## **Staticcall**

method similar to _call_, but it does not allow changing the state of the blockchain. This means that we cannot use _staticcall_ if the called function changes some state variable, for example.

* can be declared as _view_, as _staticcall_ does not allow changing the state of the blockchain.
* If we use _staticcall_ to invoke a function that changes the state of the blockchain, the function invocation will not be successful.
*   We can use _staticcall_ to read state variables. The following line of code is perfectly valid.

    ```solidity

    (, bytes memory data) = called.staticcall(abi.encodeWithSignature("number()"));

    ```


* Like _call_, _staticcall_ returns two values. A boolean, indicating the success or not of the call, and a value of type bytes, which is the return of the call.&#x20;
* As _staticcall_ does not change the state of the blockchain, it only makes sense to execute this method when we want to retrieve some value; that is, when we expect some return.

## Delegatecall <a href="#dbc1" id="dbc1"></a>

**What is the difference between `CALL` and `DELEGATECALL`?**

* `CALL`  executes the function **in the context of the contract it was defined**, while `DELEGATECALL` inherits the execution context, meaning that the function will behave as it was defined in the contract that’s using `DELEGATECALL`

```solidity
contract Called {

  uint public number;

  function setNumber(uint _number) public {
    number = _number;
  }
}
```

```solidity
contract Caller {

  uint public number;
  address public called = 0xd9145CCE52D386f254917e481eB44e9943F39138;
  
  function callSetNumber(uint _number) public {
    called.delegatecall(abi.encodeWithSignature("setNumber(uint256)",_number));
  }
}
```

* The `callSetNumber` function will call the `setNumber` function on `Called`, which will change the `number` variable.&#x20;
* However, **it will change the variable `number` of the contract `Caller`**, and not of the contract where the function is defined.
* Like _call_, _delegatecall_ also returns two values. The first is a boolean that indicates whether the transaction was successful or not, and the second is of type bytes, which is the function’s return, if any return occurs.
* In the example above, we used the same state variable name in both contracts, _number_, but it is not necessary. **The name of the variable doesn’t matter, but where it is located in storage.**
*   Using _delegatecall_, the execution that would be done in the storage of the called contract, is done in the storage of the calling contract. Thus, we need to make sure that the variables of both contracts are properly paired.

    \


    \
