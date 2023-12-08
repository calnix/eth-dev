# Payable

### A function cannot receive Ether, unless marked as `payable` .

* `address payable myAddress`
* `function myFunction() public payable{}`
* If a function/address is not marked as payable and receives Eth - it will fail.

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.1;

contract SendMoneyExample {
    uint public balanceReceived;      

    function receiveMoney() public payable {
        balancedReceived += msg.value;          //msg: global always-existing object
    }

    function getBalance() public view returns(uint) {
        return address(this).balance
    }
}
```

`uint public balanceReceived`

* public state variable -> getter function auto-created. So we can always query its value.
* _therefore, we don't need getBalance -> this is just an example._&#x20;

`balanceReceived += msg.value`&#x20;

* The msg-object is a global always-existing object containing a some information about the ongoing transaction.&#x20;

![msg object fields](<../../../.gitbook/assets/image (322).png>)

* The two most important properties are .value and .sender.&#x20;
  * .value contains the amount of Wei that was sent to the smart contract.&#x20;
  * .sender contains the address that called the Smart Contract.&#x20;

`address(this).balance`&#x20;

* A variable of the type address always has a property called .balance which gives you the amount of ether stored on that address.&#x20;
* It doesn't mean you can access them, it just tells you how much is stored there. Remember, it's all public information.&#x20;
* `address(this)` converts the Smart Contract instance to an address. So, this line essentially returns the amount of Ether that are stored on the Smart Contract itself.\
