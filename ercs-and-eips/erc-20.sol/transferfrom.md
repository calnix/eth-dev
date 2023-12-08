# transferFrom()

* Moves `amount` tokens from `sender` to `recipient` using the allowance mechanism. `amount` is then **deducted from the caller’s allowance.**
* Returns a boolean value indicating whether the operation succeeded.
* Emits a [`Transfer`](https://docs.openzeppelin.com/contracts/3.x/api/token/erc20#IERC20-Transfer-address-address-uint256-) event.

![](<../../.gitbook/assets/image (294).png>)

If A has an allowance granted by B with yourToken.sol, A has the liberty to transfer the tokens to destination address of choice. A is not limited to transferring its allowance only to its' own address.



* \---
* userA calls transferFrom() on service contract
* spender = caller of \_msgSender() ->  spender = service contract
* \_spendAllowance(from, servicecontract, amt)
  * servicecontract's allowance is decremented&#x20;
* \_transfer(from, to, amt).

\---

![](<../../.gitbook/assets/image (225).png>)

* userA calls sellTokens() - function of Service contract
  * within the scope of sellTokens(), msg.sender = userA
* msg.sender = userA, gets passed as parameter into .transferFrom()
* inside transferFrom(), Service contract calls \_msgSender() - function of YourToken.sol
  * within internal scope, Service contract calls Token contract
  * msgSender() = Service contract
  * spender = Service Contract
