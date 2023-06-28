# Destroying Smart Contracts

We can not erase information from the Blockchain, but we can update the _current state_ so that you can't interact with an address anymore _going forward_.&#x20;

Everyone can always go back in time and check what was the value on day X, but, once `selfdestruct` is called, you can't interact with a Smart Contract anymore.

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

    function destroyContract(address payable _to) public {
        require(owner == msg.sender, "Only ownder can destroy");
        selfdestruct(_to);

    }
}
```

`selfdestruct` function ->  takes one argument, an address. `selfdestruct(beneficiaryAddress)`&#x20;

When `selfdestruct` is called, all remaining funds on the address of the Smart Contract are transferred to that address.

{% hint style="warning" %}
You can still send ETH via "sendMoney" function to the SC. \


There won't be an error. Internally you are sending Ether to an address. Nothing more - there no pre-checks to confirm if selfdestruct has not been called. You locked one Ether at the Smart Contract address for good.

Essentially selfDestruct simply permanently deactivates it. You can review the block before to examine the SC before destruction.
{% endhint %}

### CREATE2 Op-Code

Since the [CREATE2](https://eips.ethereum.org/EIPS/eip-1014) Op-Code was introduced, you can pre-compute a contract address.

Without CREATE2, a contract gets deployed to an address that is computed based on _your_ address + your _nonce_. That way it was guaranteed that a Smart Contract cannot be re-deployed to the same address.

With the CREATE2 op-code you can _instruct_ the EVM to place your Smart Contract on a specific address. Then you could call selfdestruct(), thus remove the source code. Then re-deploy a different Smart Contract to the same address.

This comes with several implications: when you see that a Smart Contract includes a `selfdestruct()` then simply be careful. Those implications will become more and more apparent as you progress through the course, especially when we talk about the ERC20 Token allowance. At this stage it is too early to discuss them all, but if you want to read on about it, checkout [this article](https://medium.com/@jason.carver/defend-against-wild-magic-in-the-next-ethereum-upgrade-b008247839d2).
