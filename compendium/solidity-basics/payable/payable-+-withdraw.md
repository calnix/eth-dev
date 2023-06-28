---
description: Adding a dumb withdraw function withdrawMoney()
---

# Payable + withdraw

```solidity
// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.1;

contract SendMoneyExample {
    
    //public state variable -> getter function auto-created.
    uint public balanceReceived;                

    function receiveMoney() public payable {
        balanceReceived += msg.value;          //will not account for withdrawls as it only increments.

    function getBalance() public view returns(uint) {
        return address(this).balance;
    }

    function withdrawMoney() public {
        //to = payable(msg.sender) works from 0.8.0
        address payable to = payable(msg.sender);  
        to.transfer(getBalance());
    }
}
```

* `address to = msg.sender;`
  * `to` is an address, which we set to take the value of the sender's wallet address (`msg.sender`).&#x20;
  * Sender in the sense of transaction hash. Someone else is creating and sending this transaction: "from".  (fella who called the SC).
* if we want to send money to this address, we need to make it explicitly payable.
  * `address payable` to = `payable`(msg.sender);
* `transfer` is another property of an address object.
  * here we have stupidly opted to send all the SC's ether back to the sender/caller's address.

{% hint style="success" %}
If you want to send ether to an address, the address must be of **type address payable**. That is why you are **casting msg.sender as payable.** What is payable is not the amount, but the address.

**payable(msg.sender)** works from 0.8.0\
[https://ethereum.stackexchange.com/questions/64108/whats-the-difference-between-address-and-address-payable](https://ethereum.stackexchange.com/questions/64108/whats-the-difference-between-address-and-address-payable)
{% endhint %}

{% hint style="info" %}
Notice we have no need to declare storage/memory for to, when we declare and initialize it within a function

address is a value type -> no need to specify data location.
{% endhint %}

### Adding withdrawTo

```solidity
// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.1;

contract SendMoneyExample {

    uint public balanceReceived;                //public state variable -> getter function auto-created.

    function receiveMoney() public payable {
        balanceReceived += msg.value;          //will not account for withdrawls as it only increments.
    }
    
    function getBalance() public view returns(uint) {
        return address(this).balance;
    }

    function withdrawMoney() public {
        address payable to = payable(msg.sender);
        to.transfer(getBalance());
    }
    
    function withdrawMoneyTO(address payable _to) public {
        to.transfer(getBalance());
    }
    
}
```

in withdrawTO, we declare to as address and payable in the function parameter, so there is no need to do so in the body.

function will expect an address to be supplied on call, following which it will transfer all the SC's balances to the address -> not locked to sender's address. &#x20;

### Time-locked Withdrawals

`block.timestamp` is a global object.&#x20;

* contains the timestamp when a block was mined.
* Not necessarily the _current timestamp_ when the execution happens. It might be a few seconds off. But it's still enough to do some locking.

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.1;
contract SendMoneyExample {

uint public balanceReceived;               
uint public lockedUntil;

function receiveMoney() public payable {
    balanceReceived += msg.value;
    lockedUntil = block.timestamp +1 minutes;         

function getBalance() public view returns(uint) {
    return address(this).balance;
}

function withdrawMoney() public {
    address payable to = payable(msg.sender);
    to.transfer(getBalance());
}

function withdrawMoneyTO(address payable _to) public {
    if(block.timestamp > lockedUntil){
        _to.transfer(getBalance());
    }
        
}
```

* added uint public lockedUntil;
* added to function receiveMoney(): lockedUntil = block.timestamp +1 minutes;
* added to function withdrawMoneyTO: if condition to check time

Now, after sending money, click "withdrawMoney" - and nothing happens. The Balance stays the same until 1 Minute passed since you hit "receiveMoney".

{% hint style="danger" %}
call is the recommended way of sending ETH (do no use send or transfer)
{% endhint %}

## Transfer v send v call

In a failure case, the **transfer** function **reverts**.

**Send** is similar to transfer. But if the payment fails, it **will not revert.** Instead, it returns false. The failure handling is left to the calling contract. (code continues execution)

For both send and transfer 2300 gas is forwarded to the receiving contract. Small amount sufficient to trigger an event for logging.

### Problems with send() and transfer()

Both functions were considered the go-to solution for making payments to EOAs or other smart contracts. But since the amount of gas they forward to the called smart contract is very low, the called smart contract can easily run out of gas. This would make the payment impossible.

The problem here is that even if the receiving contract is designed carefully not to exceed the gas limit, future changes in the gas costs for some opcodes can break this design.

Therefore, the recommended way of sending payments is the function call().

{% hint style="warning" %}
After the Istanbul hardfork `send` and `transfer` methods have been deprecated.&#x20;
{% endhint %}

## call()

call is the recommended way of sending ETH from a smart contract.

#### A simple call-statement looks like that:

```solidity
(bool success, bytes memory data)= receivingAddress.call{value: 100}("");
```

Let’s have a look on the right side. The value specifies how many wei are transferred to the receiving address. In the round brackets, we can add additional data like a function signature of a called function and parameters.&#x20;

If nothing is given there, the fallback() function or the receive() function is called. &#x20;

The return value for the `call` function is a **tuple of a boolean and bytes array**.&#x20;

* boolean is the success or failure status of the call&#x20;
* bytes array has the return values of the receiving contract's function called which need to be decoded.
* [https://kushgoyal.com/ethereum-solidity-how-use-call-delegatecall/](https://kushgoyal.com/ethereum-solidity-how-use-call-delegatecall/)

### &#x20;A typical implementation

```solidity
// example
(bool success, ) = msg.sender.call{value:amt}("");
require(success, "Transfer failed.");
```

If the transfer fails, `success` is assigned the value `false`, that means the `require` statement would evaluate the false and fail, therefore reverting everything does so far back to original state.

code cannot progress forth unless `success` is `true`.

using _**call**_, one can also trigger other _**functions**_ defined in the contract and send a fixed amount of gas to execute the function. The transaction status is sent as a **boolean** and the return value is sent in the data variable.

```
(bool sent, bytes memory data) = _to.call{gas :10000, value: msg.value}("func_signature(uint256 args)");
```

## **Issues with call()**

With call(), the EVM transfers all gas to the receiving contract, if not stated otherwise. This allows the contract to execute complex operations at the expense of the function caller.

Another issue is that it allows for so-called re-entrancy attacks. This means that the receiver contract calls the function again where the call() statement is given. If the sender contract is improperly coded, it can result in draining larger amounts of funds from it than planned. This issue requires more awareness by the contract authors.

## Calling payable functions

{% tabs %}
{% tab title="Vault.sol" %}
```
weth.deposit{value: 1 ether}();
```
{% endtab %}

{% tab title="WETH.sol" %}
```solidity
function deposit() public payable {
    balanceOf[msg.sender] += msg.value;
    emit Deposit(msg.sender, msg.value);
}
```
{% endtab %}
{% endtabs %}

From Vault we call the payable function deposit, and pass msg.value within {value: 1 ether}
