# Adding Events

[https://ethereum-blockchain-developer.com/040-shared-wallet-project/07-add-events/](https://ethereum-blockchain-developer.com/040-shared-wallet-project/07-add-events/)\


```solidity
/SPDX-License-Identifier: MIT
pragma solidity ^0.8.1;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol";

contract Allowance is Ownable{

    mapping (address=> uint) allowance;
    event AllowanceChanged(address indexed _forwho, address indexed _byWhom, uint _oldAmount, uint _newAmount);
    
    //check if you are the owner
    function isOwner() internal view returns(bool){
        return owner() == msg.sender;
    }

    //set allowance
    function setAllowance(uint _amt, address _who) public onlyOwner{
        uint _oldAmount = allowance[_who];
        allowance[_who] += _amt;
        uint _newAmount = allowance[_who];
        
        emit AllowanceChanged(_who, msg.sender,_oldAmount,_newAmount);
    }

    modifier ownerOrBeneficiary(uint _amt) {
        require(isOwner() || _amt <= allowance[msg.sender], "you are not allowed");
        _;
    }

    function reduceAllowance(uint _amt, address _who) internal {
        allowance[_who] -= _amt;
    }

}
```

#### Adding events to Sharedwallet

```solidity
contract SharedWallet is Allowance{

    event receivedMoney(address indexed _fromwho, uint _amt);
    event withdrawnMoney(address indexed _bywho, uint _amt);

    receive () external payable{    //receive eth
        emit receivedMoney(msg.sender, msg.value);
        allowance[msg.sender] += msg.value;
    }

    
    function withdraw(uint _amt, address payable _to) public ownerOrBeneficiary(_amt) {
        require(_amt <= address(this).balance, "There are not enough funds");
        if(isOwner() == false){
            reduceAllowance(_amt,msg.sender);
        }
        emit withdrawnMoney(msg.sender, _amt);
        _to.transfer(_amt);

    }

}
```

