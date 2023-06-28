# Smart Contract Life-cycle

How to start, stop, update and destroy a smart contract.

### Insecure Example

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.1;

contract StartStopUpdateExample {

    function sendMoney() public payable {

    }

    function withdrawAllMoney(address payable _to) public {
        _to.transfer(address(this).balance);
    }
}
```

![](<../../.gitbook/assets/image (47).png>)

* with sendMoney() we can send the SC some ETH
* with withdrawAllMoney(), **anyone** can input their address and withdraw all balances.

### Secure Example

We will secure this SC by ensuring that the person interacting with `withdrawAllMoney` is the same as the one who deployed the Smart Contract.

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.1;

contract StartStopUpdateExample {
    
    // set ownership
    address public owner;
    
    constructor(){
        owner = msg.sender;
    }

    function sendMoney() public payable {
    }

    function withdrawAllMoney(address payable _to) public {
        require(owner == msg.sender, "You are not the the owner - cannot withdraw!");
        _to.transfer(address(this).balance);
    }

}
```

* create an address state variable to hold the owner's address.
* use constructor, which will run on deployment, to assign `owner` with the deployer's address.
  * whomever deploys is deemed to be the owner automatically.
* add conditional check, `require` statement, to withdraw function
  * `require` is like `if`, if this condition is true -> proceed. else fails.
  * require checks if the person initiating the withdraw function is indeed owner.
  * if true, balances transferred.
  * If require evaluates to false it will stop the transaction, roll-back any changes made so far and emit the error message as String.

![](<../../.gitbook/assets/image (137).png>)

{% hint style="info" %}
Note, the msg.sender would be different for the constructor and the withdraw as these are two different interactions with 2 different messages.&#x20;

msg of constructor -> emitted on whomever deployed

msg of withdraw -> emitted on whomever calling withdraw()
{% endhint %}

#### An alternative&#x20;

An alternative way is require(\_to == owner, "...'), this will only allow withdrawals made to the owner/deployer's address.

The above allows the owner to withdraw to an address of choice, different from his deployment wallet.&#x20;

{% hint style="info" %}
Other ownable models:

[https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol)
{% endhint %}
