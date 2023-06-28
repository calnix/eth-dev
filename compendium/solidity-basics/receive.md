# Receive

### Receive()

* It cannot have arguments.
* It cannot return data.&#x20;
* It has `external` visibility and is marked `payable`.

The `receive` method is used as a fallback function in a contract and is called when ether is sent to a contract _**with no calldata**_. If the `receive` method does not exist, it will use the `fallback` function.

```solidity
//SPDX-License-Identifier: MIT
pragma solidity 0.8.3;

contract FunctionsExample {

    mapping(address => uint) public balanceReceived;

    function receiveMoney() public payable {
        assert(balanceReceived[msg.sender] + msg.value >= balanceReceived[msg.sender]);
        balanceReceived[msg.sender] += msg.value;
    }

    function withdrawMoney(address payable _to, uint _amount) public {
        require(_amount <= balanceReceived[msg.sender], "not enough funds.");
        assert(balanceReceived[msg.sender] >= balanceReceived[msg.sender] - _amount);
        balanceReceived[msg.sender] -= _amount;
        _to.transfer(_amount);
    }

    receive() external payable{
        receiveMoney();
    } 

}
```

* `receive() external payable` — for empty calldata
* `fallback() external payable` — when no other function matches (not even the receive function). Optionally `payable`.

**Links:**

* [https://blog.finxter.com/what-is-payable-in-solidity/](https://blog.finxter.com/what-is-payable-in-solidity/)
