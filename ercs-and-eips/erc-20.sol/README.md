---
description: >-
  https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol
---

# ERC-20.sol

## Variable Definitions:

### \_balances mapping

![](<../../.gitbook/assets/image (53).png>)

* tracks token holdings of each address
* **private:** it can **only** be accessed within this contract.&#x20;
  * Contracts deriving from it, cannot access this mapping.

### \_allowances mapping

![](<../../.gitbook/assets/image (188).png>)

\--> mapping(address => \[address, uint])

* tracks token allowances of each address.
* for wallet A, what is the allowance set for a specific address, e.g. UniSwap,&#x20;
* Supply parameters as such: \_allowances\[owner]\[spender];

![](https://lh4.googleusercontent.com/uZ13Oe1BmlRigHqJJROlkxlhdaTwcy7vAc6Oel-8hzZKzy79xBhscrbZVi9r6Dzzggfg0gIxgvbdF58W0Gxa7aRKgfh0rpPWzozXzb8jbAyaGe5czjknCoyOPRIoNtobf\_ROEoI)

Note that anyone can query any address’ allowance, as all data on the blockchain is public. It is important to understand that allowances are “soft”, in that both individual and cumulative allowances can exceed an address’ balance.&#x20;

In the approval table shown above, for example, the holder 0x2299…3ab7 has approved a transfer of up to 500 tokens but their balance as seen previously is only 90 tokens. Any contract using allowance() must additionally consider the balance of the holder when calculating the number of tokens available to it.
