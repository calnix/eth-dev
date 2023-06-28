# Project #2: Basic Vault

## Objective: multi-user safe

1. This is a single-token Vault that holds a pre-specified ERC-20 token.
2. Users can send tokens to the Vault contract.
3. Vault contract records the user's token deposits.
4. Users can withdraw their tokens only to the address they deposited from.

### Contracts

* ERC20Mock.sol
* BasicVault.sol

### Vault contract

1. Deposit: After approval (handled by front-end), token is deposited via transferFrom.
2. Withdraw: Token is returned, and the Vault updates its user records.

> Vault contract should import the IERC20 interface, take the Token address in the Vault constructor, and cast it into an IERC20 state variable

### ERC2Mock contract

1. Public, unrestricted mint() function: anyone should be able to mint
2. Import [https://github.com/yieldprotocol/yield-utils-v2/blob/main/src/mocks/ERC20Mock.sol](https://github.com/yieldprotocol/yield-utils-v2/blob/main/src/mocks/ERC20Mock.sol)
3. Come up with your own fun token name

### Additional Details

1. Add full NatSpec: the contract as well as every event and function should have complete NatSpec.

* contracts should have @title, @notice, @dev, @author.
* functions should have @notice, @dev, @param, @return.
* events should have @notice (no need param or return). [https://docs.soliditylang.org/en/v0.5.10/natspec-format.html](https://docs.soliditylang.org/en/v0.5.10/natspec-format.html)

2. Implement CI so when you push to gh, your tests are run automatically:&#x20;

* [https://gist.github.com/clifton/b5ee5286bb229281fb31d7c4b15e6f31](https://gist.github.com/clifton/b5ee5286bb229281fb31d7c4b15e6f31)
* [https://book.getfoundry.sh/config/continous-integration.html](https://book.getfoundry.sh/config/continous-integration.html)

### Testing

* Vault contract should be fully tested.
* No need to test ERC20Mock. Assume it works as intended.

#### Testing Guidelines

* All state variable changes in the contracts that you code.
* All state variable changes in other contracts caused by calls from contracts that you code.
* All require or revert in the contracts that you code.
* All events being emitted.
* All return values in contracts that you code.

## Blueprint

```solidity
contract BasicVault {
    
    // STATE VARS: 
    // token 
    // mapping to serve as balances registry
    // events 
    
    // constructor function
    
    // deposit function
    
    // withdraw function 

}

```
