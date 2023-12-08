# Common Code Contract

We will split the allowance checks into a seperate contract, like so:



```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.1;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol";

contract Allowance is Ownable{

    mapping (address=> uint) allowance;
    
        //check if you are the owner
    function isOwner() internal view returns(bool){
        return owner() == msg.sender;
    }
    modifier ownerOrBeneficiary(uint _amt) {
        require(isOwner() || _amt <= allowance[msg.sender], "you are not allowed");
        _;
    }

    function reduceAllowance(uint _amt, address _who) internal {
        allowance[_who] -= _amt;
    }
}

contract SharedWallet is Allowance{

    receive () external payable{    //receive eth
        allowance[msg.sender] += msg.value;
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

Allowance inherits from Ownable, and Shared Wallet inherits from Allowance (+ Ownable indirectly).

When deploying, you would deploy the SharedWallet contract.

![](<../../.gitbook/assets/image (278).png>)
