# Project #4: Fractional Wrapper

### Objectives

* Users can send a pre-specified erc-20 token (underlying) to an ERC20 contract(Fractional Wrapper).
* Fractional Wrapper contract issues a number of Wrapper tokens to the sender, equal to the deposit multiplied by a fractional number, called exchange rate. Exchange rate is set by the contract owner.
* This number is in the range of \[0, 1e18], and available in increments of 10\*\*-27. (ray).
* At any point, a holder of Wrapper tokens can burn them to recover an amount of underlying equal to the amount of Wrapper tokens burned, divided by the exchange rate.
* This Fractional Wrapper can conform to the [ERC4626 specification](https://eips.ethereum.org/EIPS/eip-4626).&#x20;

#### Process

1. User sends DAI to FWrapper.
2. User receives wDAI, (wDAI = DAI \* ex\_rate)
3. Exchange rate set by FWrapper owner
4. Ex\_rate is has decimal precision of 10\*\*27 precision
5. User can liquidate and get back underlying DAI, (wDAI is burnt) ---> dai\_qty = wDAI/ex\_rate

### Contracts

1. DAIToken - underlying
2. FractionalWrapper - "vault w/ ex\_rate"
3. Ownable.sol - for onlyOwner modifier, applied on setExchangeRate

Both contracts are ERC20Mock, to issue tokens. FractionalWrapper must conform to ERC4626 specification.

* all methods must be implemented
* implementation of convert\* and preview\* will be identical in this case (no need to calculate some time-weighted average for convert\*).

\
