# Project #3: ERC20+ETH Wrapper

## ERC20 Wrapper

Users can exchange ERC20 TokenA for ERC20 TokenB via a contract called Wrapper. Wrapper contract will do a 1 for 1 exchange. (Similar to WETH contract)

### Objectives

1. Users can send a pre-specified ERC20 token to a Wrapper contract (also an ERC20).
2. Wrapper contract issues an equal number of Wrapper tokens to the sender.
3. Holder of wrapper tokens can claim their original ERC20 tokens, by burning wrapped tokens.

_The functions used will be similar to those in the Vault, but you will notice that now we use tokens as a system of record._

### Contracts

* Create a mock token (inherit ERC20.sol)
* ERC20Wrapper.sol&#x20;

_ERC20Wrapper.sol will **store** mock tokens and issue out wrapper tokens for the same amount._

### Testing

Focus will be on the ERC20Wrapper contract.

Testing states:

*

## Stretch Goals

Ether is not an ERC-20 token. Code a Wrapper that takes Ether as it's currency.

### Contract

ETHWrapper.sol

### Testing

StateZeroTest: (user has 1 ETH)

* Test if user can send ether, and receive the corresponding amount of wrapped tokens. (testReceive)

StateUserWithWrappedETHTest: (user has 0.5 ETH, 0.5 WETH)

* Test if user can receive their ETH back, by calling unwrap (testUnwrap)

## Deployment

