# Pausing Smart Contracts

A Smart Contract cannot be "paused" on a protocol-level on the Ethereum Blockchain. \
But we can add conditional logic to our to pause it.

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.1;

contract StartStopUpdateExample {
    
    address public owner;
    bool public is_paused;

    constructor(){
        owner = msg.sender;
    }

    function sendMoney() public payable {

    }

    function setPaused(bool _paused) public {
        require(owner == msg.sender, "Only ownder can set pause state");
        is_paused = _paused;
    }

    function withdrawAllMoney(address payable _to) public {
        require(is_paused == false, "Contract paused");
        require(owner == msg.sender, "You are not the the owner - cannot withdraw!");
        _to.transfer(address(this).balance);
    }

}
```

* add is\_paused as a public state variable.
* add function setPaused() with require
  * only deployer can update paused state.
* add require(is\_paused) check to withdraw money
  * if paused == false, function is active.



\
