---
description: https://github.com/calnix/ERC20-Wrapper
---

# #3 ERC20+ETH Wrapper

* Problem Statement: [https://github.com/yieldprotocol/mentorship2022/issues/3](https://github.com/yieldprotocol/mentorship2022/issues/3)
* Github: [https://github.com/calnix/ERC20-Wrapper](https://github.com/calnix/ERC20-Wrapper)

## ERC20 Wrapper

Users can exchange ERC20 TokenA for ERC20 TokenB via a contract called Wrapper. Wrapper contract will do a 1 for 1 exchange. (Similar to WETH contract)

### Objectives

1. Users can send a pre-specified ERC20 token to a Wrapper contract (also an ERC20).
2. Wrapper contract issues an equal number of Wrapper tokens to the sender.
3. Holder of wrapper tokens can claim their original ERC20 tokens, by burning wrapped tokens.

### Contracts

QTMToken.sol (inheriting ERC20Mock.sol) ERC20Wrapper.sol (Wrapped contract that can be generically applied to any ERC20 token)

\*\* ERC20Wrapper.sol will **store** QTM tokens and issue out wQTM for the same amount.

### Testing

Focus will be on the ERC20Wrapper contract.

StateQTMmintedTest: (user has minted QTM tokens and approved then for spending by the Wrapper contract)

* if transfer fails, deposit should revert (testWrapRevertsIfTransferFails)
* deposit tokens into vault (testWrap)

StateQTMWrapped: (user holds wQTM - has exchanged QTM for wQTM)

* cannot unwrap if transfer fails (testUnwrapRevertsIfTransferFails)
* cannot unwrap more than that which was wrapped (testCannotUnwrapExcessOfWrapped)
* Fuzz testing withdrawals (testUnwrap)

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

Arbitrum-Rinkeby

* Use public RPC: [https://rinkeby.arbitrum.io/rpc](https://rinkeby.arbitrum.io/rpc)
