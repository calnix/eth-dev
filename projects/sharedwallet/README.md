---
description: >-
  https://ethereum-blockchain-developer.com/040-shared-wallet-project/03-open-zeppelin/
---

# SharedWallet

* anyone can deposit.&#x20;
* owner can withdraw everything&#x20;
* assigned beneficiary can withdraw based on allowance variable

This is what we cooked up initially. We coded our own basic owner  functionality with the variable and require statements.

```solidity
//SPDX-License-Identifier: MIT 
pragma solidity ^0.8.1; 

contract SharedWallet {
    address owner;
    uint balance
    address receiver;
    uint allowance;

    constructor(){
        owner = msg.sender;         //attribute owner on deployment
    }
    
    function deposit() payable public{
        balance += msg.value;
    }
    // OR :
    receive () external payable{
        balance += msg.value;
    }
    
    // centralising permissions:
    modifier onlyOwner(){
        require(msg.sender == owner, "you are not the owner");
        _;
    }
    
    function withdraw(uint _w_amt, address payable _to) public onlyOwner {
        require(_w_amt <= balance, "Not enough balance");
            balance -= _w_amt;
            _to.transfer(_w_amt);
    }
    
    function receiveAllowance(uint _w_amt, address payable _to) public {
        require(msg.sender == receiver, "must be beneficiary");
        require(_w_amt <= allowance, "Cannot exceed allowance or balance");
            balance -= _w_amt;
            _to.transfer(_w_amt);
    }
}
```

However, OpenZeppelin has already built a comprehensive ownership functionality, allowing for transfers, renouncement and much more. We can simply re-use it, by importing it.&#x20;

Our modifier onlyOwner should be removed as Ownable.sol also defines onlyOwner (line 42). This will cause an error, with Solidity expecting the `override` specifier.

&#x20;

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.1;

// for Owner functionality - no need to built youself
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol";

contract SharedWallet is Ownable{   //to extend base Ownable

    uint balance;
    address receiver;
    uint allowance;

    receive () external payable{    //receive eth
        balance += msg.value;
    }
    
    //check if you are the owner
    function isOwner() internal view returns(bool){
        return owner() == msg.sender;
    }

    function withdraw(uint _w_amt, address payable _to) public onlyOwner {
        require(_w_amt <= balance, "Not enough balance");
            balance -= _w_amt;
            _to.transfer(_w_amt);

    }

    function receiveAllowance(uint _w_amt, address payable _to) public {
        require(msg.sender == receiver, "must be beneficiary");
        require(_w_amt <= allowance, "Cannot exceed allowance or balance");
            balance -= _w_amt;
            _to.transfer(_w_amt);
    }
}
```

The latest OpenZeppelin contract does not have an isOwner() function anymore, so we have to create our own.&#x20;

Note that the owner() is a function from the Ownable.sol contract.
