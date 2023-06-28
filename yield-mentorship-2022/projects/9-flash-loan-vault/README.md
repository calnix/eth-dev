---
description: ERC 3156 & ERC 4626 compliant
---

# #9 Flash Loan Vault

* Problem Statement: [https://github.com/yieldprotocol/mentorship2022/issues/9](https://github.com/yieldprotocol/mentorship2022/issues/9)
* Github: [https://github.com/calnix/Flash-Loan-Yield-Bearing-Vault](https://github.com/calnix/Flash-Loan-Yield-Bearing-Vault)

## Objective

Refactor the Fractional Wrapper from assignment #4 into a Proportional Ownership Flash Loan Server, conforming to specifications ERC3156 and ERC4626.

1. Refactor the Fractional Wrapper into an ERC3156 Flash Loan Server: https://eips.ethereum.org/EIPS/eip-3156
2. The underlying is the token available for flash lending -> e.g. DAI | wrapper token = yvDAI
3. Fee is a constant 0.1%
4. Make the contract a proportional ownership contract
5. When users deposit underlying, they get a proportion of wrapper tokens, based on the following:

> wrapperMinted / wrapperSupply == underlyingDeposited / underlyingInWrapper

### What does this mean?

Essentially, the Fractional wrapper operates as per usual; accepting deposits, withdrawals and so forth as specified in ERC4626.&#x20;

Additionally, it can now offer flashloans, conforming to ERC3156. The flashlending token will be  the underlying token that the FractionWrapper operates upon.

#### For example:&#x20;

Users can deposit DAI, which is the underlying, for which they will receive yvDAI (wrapped tokens) as per:

> wrapperMinted = (underlyingDeposited \* wrapperSupply) / underlyingInWrapper

* Other users can flash borrow DAI that has been deposited, for whatever purpose they have in mind.&#x20;
* They will be charged a fee of 0.1% for this service.&#x20;
* In this way, the Vault accumulates fees, which can then be partially passed on to depositors as yield, while the rest is retained as revenue.

### Exchange rate is dynamic&#x20;

Initial deployment will be an edge case, as supply of tokens will be 0. So we begin with a peg.

Assume at t=0 (s=0), exchange rate is 1 -> 1 DAI : 1 yvDAI

Deployer deposits DAI in to vault to get the ball rolling, and receives wrapper tokens as per the peg.

> Deposit 1000 DAI, receive 1000 yvDAI tokens

After this initial deposit, the exchange rate still holds at the peg.

The peg "moves" or is altered based on the yield generated from flash loans.&#x20;

Since there is a fee levied for the flash loan service, supply of underlying tokens will increase and this will result in the exchange rate float away from peg.



## Flash Loan flow

1. Borrower contract executes flashBorrow():
   * total repayment amount (fee + amount) is calculated and approved as allowance for lender to claw back via transferFrom()
   * lender.flashloan() is called, initiating the flashloan service on the lender contract.
2. flashLoan() checks the following:
   * Tokens that was being requested for flash loan are supported & calculates fee to be charged
   * Transfers the loan amount to the receiver (via .transfer)
   * Call back to receiver: calls receiver.onFlashLoan() which does the following:
     * verifying the sender is the correct lender
     * verifying the initiator for the flash loan was actually the receiver contract
     * returns keccak256("ERC3156FlashBorrower.onFlashLoan")
   * Finally, amount+fee is transferred from the receiver to the lender (via .transferFrom)
   * returns true on success.
