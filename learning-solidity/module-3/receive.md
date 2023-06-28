# Receive

### Receive()

Receive is a special function in Solidity that allows contracts to receive funds from external accounts.

* It cannot have arguments.
* It cannot return data.&#x20;
* It has `external` visibility and is marked `payable`

**Examples**

**Use `receive()` to accept ether:**

```solidity
contract MyContract {    // Receive Ether    
    receive() external payable { // Do something with the received ether }
}
```

**The receive function can also be used to accept tokens:**

```solidity
import "./token.sol"; // A standard ERC20 token 

contract MyContract {    
    Token token; 
    
    constructor(address _tokenAddress) {        
        token = Token(_tokenAddress);    
    } 
    
    // Receive ERC20 tokens    
    receive() external payable {        
        uint256 amount = msg.value;        
        token.transferFrom(msg.sender, address(this), amount);    
    }
}
```
