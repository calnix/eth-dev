# Payable

## Payable Function

Payable is a keyword in Solidity that enables a function to receive Ether. When declaring a function as payable, it indicates that the function is able to receive Ether from external calls.

* If a function is not marked as payable, transactions sending ether will fail.

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.1;

contract SendMoneyExample {
    uint public balanceReceived;      

    function receiveMoney() public payable {
        balancedReceived += msg.value;      //msg: global always-existing object
    }

}
```

### Calling payable functions

Say you want to make a call to **another contract's payable function** and transfer ETH:

```css
targetAddress.someFunction{ value: amount }(arg1, arg2, arg3)
```

* value: amount of eth to send contract

{% tabs %}
{% tab title="WETH.sol" %}
```solidity
function deposit() public payable {
    balanceOf[msg.sender] += msg.value;
    emit Deposit(msg.sender, msg.value);
}
```
{% endtab %}

{% tab title="Vault.sol" %}
```solidity
weth.deposit{value: 1 ether}();
```
{% endtab %}
{% endtabs %}

* From Vault, we call the payable function `deposit`, and pass msg.value within the `value` field
* [https://ethereum.stackexchange.com/questions/60279/how-to-set-msg-value-in-solidity-function-call](https://ethereum.stackexchange.com/questions/60279/how-to-set-msg-value-in-solidity-function-call)

## Payable Address

* If you want to send ether to an address, the address must be of type **`address payable`**.&#x20;

**Example**

Sending eth to a user

```solidity
contract ExampleContract {    
  address public owner;    

  constructor() public {       
     owner = msg.sender;    
  }   

  function withdrawFunds() public payable {        
    address payable recipient = payable(msg.sender);  
    recipient.transfer(msg.value);     
  }
}
```

* Regular addresses are not able to receive ethereum, and need to be casted as payable addresses to receive ETH.
* That is why you are **casting msg.sender as payable.**&#x20;

{% hint style="success" %}
[https://ethereum.stackexchange.com/questions/64108/whats-the-difference-between-address-and-address-payable](https://ethereum.stackexchange.com/questions/64108/whats-the-difference-between-address-and-address-payable)
{% endhint %}

### Built-in methods

* You can use `.transfer(..)` and `.send(..)` on `address payable`, but not on `address`.
* You can use a low-level `.call{value: ..}(..)` on both `address` and `address payable`, even if you attach value.
* `.call` is the recommended way of sending ether from a smart contract.

#### Problems with send() and transfer()

* Both functions were considered the go-to solution for making payments to EOAs or other smart contracts. But since the amount of gas they forward to the called smart contract is very low, the called smart contract can easily run out of gas. This would make the payment impossible.
* The problem here is that even if the receiving contract is designed carefully not to exceed the gas limit, future changes in the gas costs for some opcodes can break this design.
* Therefore, the recommended way of sending payments is `call`.

{% hint style="warning" %}
After the Istanbul hardfork `send` and `transfer` methods have been deprecated.&#x20;
{% endhint %}

## call()

#### A simple call-statement looks like that:

```solidity
(bool success, bytes memory data) = receivingAddress.call{value: 100}("");
```

* `value` specifies how many wei are transferred to the receiving address. I
* In the round brackets, we can add additional data like a function signature and its parameters.&#x20;
* If nothing is passed there, the `fallback` function or the `receive` function is called. &#x20;

The return value for the `call` function is a **tuple of a boolean and bytes array**.&#x20;

* boolean is the success or failure status of the call&#x20;
* bytes array has the return values of the receiving contract's function called which need to be decoded.

{% hint style="info" %}
[https://kushgoyal.com/ethereum-solidity-how-use-call-delegatecall/](https://kushgoyal.com/ethereum-solidity-how-use-call-delegatecall/)
{% endhint %}

### A practical implementation

```solidity
// example
(bool success, ) = msg.sender.call{value:amt}("");
require(success, "Transfer failed.");
// ... other code ... //
```

* If the transfer fails, `success` is assigned the value `false`,&#x20;
* `require` statement would evaluate the `false` and fail, therefore reverting the transaction.
* If `success` is `true` code progresses onward.

#### Function call

Using **call**, one can also trigger other **functions** defined in the contract and send a fixed amount of gas to execute the function.&#x20;

```solidity
(bool sent, bytes memory data) = _to.call{gas :10000, value: msg.value}("func_signature(uint256 args)");

```

