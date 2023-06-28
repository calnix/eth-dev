# Multiple Beneficiaries

We want to have multiple beneficiaries, as opposed to a single one before.

So we remove the state variables `receiver` and `allowance`  and introduce `mapping (address=> uint) allowance;`

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.1;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol";

contract SharedWallet is Ownable{

    mapping (address=> uint) allowance;
  
    //check if you are the owner
    function isOwner() internal view returns(bool){
        return owner() == msg.sender;
    }

    receive () external payable{    //receive eth
        allowance[msg.sender] += msg.value;
    }

    modifier ownerOrBeneficiary(uint _amt) {
        require(isOwner() || _amt <= allowance[msg.sender], "you are not allowed");
        _;
    }

    function reduceAllowance(uint _amt, address _who) internal {
        allowance[_who] -= _amt;
    }

    function withdraw(uint _amt, address payable _to) public ownerOrBeneficiary(_amt) {
        require(_amt <= address(this).balance, "There are not enough funds");
        if(isOwner() == false){
            reduceAllowance(_amt,msg.sender);
        }
        _to.transfer(_amt);

    }
}
```

Instead of having 2 withdraw functions, one for owner and one for beneficiary, we will have one withdraw function that can check ownership condition and has a modifier.

#### modifier ownerOrBeneficiary

ensures that only either the owner or a beneficiary can call the function.&#x20;

For the second condition in the require statement.

* We don't have a locked list of beneficiary addresses to compare against, so we will simply check if said address has a balance that can meet requested withdrawal amount.

#### function withdraw(...)

The nested if condition checks if its a beneficiary, if so, `reduceAllowance()` is executed - this will update the balance of beneficiary.&#x20;

{% hint style="info" %}
We are being lazy with the withdrawal function for owner. In that, when the owner withdraws, balances are not updated. Also there is the question of accounting, from which addresses is he withdrawing from. Ignore this hole as it is a learning exmaple.&#x20;
{% endhint %}

#### function reduceAllowance

made internal arbitrarily for inheritance stuff later.
