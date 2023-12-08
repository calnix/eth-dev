# transfer()

### transfer()

![](https://cdn-images-1.medium.com/max/800/1\*ATY-2GJJl4\_yUHIZlbQJag.png)

transfer() is called on when we want to initiate a transfer of tokens **from ourselves (the function caller) to another address.**

* Moves `amount` tokens from the caller’s account to `recipient`.
* Returns a boolean value indicating whether the operation succeeded.

One usage of this would be in a dApp Vendor contract that sells ''yourToken'' for a price. The Vendor will transfer tokens from itself to the user when a purchase it made. For example, userA wants to buy tokens from a Vendor (service contract).

![](<../../.gitbook/assets/image (311).png>)

* userA calls buyTokens() which is a function of the service contract (Vendor.sol), and sends some amount of ETH.
* after inventory checks clear, service contract calls token contract (yourToken.sol), instructing a transfer(_to =_ msg.sender, tokenQty)
  * msg.sender -> userA,&#x20;
  * because within the scope of buyTokens(), caller was userA.
  * this gets passed on as a parameter object.
  * service contract is telling token contract that it wants to transfer tokens **to userA.**
* Inside the scope of transfer(msg.sender, tokenQty),
  * \_msgSender() -> service contract
  * because from Context.sol's perspective, \_msgSender was called by service contract.&#x20;
  * owner = service contract
  * _transfer(owner, to, amount) -> \_transfer(Vendor, userA, amt)_

transfer() calls two other functions, `_msgSender()` and `_transfer()`.

1. **\_msgSender()**&#x20;

\_msgSender() is inherited from Context.sol, which is an abstract contract, defined as follows:

![](<../../.gitbook/assets/image (266).png>)

From `Context.sol`'s perspective, the function caller is the service contract, so msg.sender within this scope returns Vendor's address to be captured in `owner`.&#x20;

From documentation:&#x20;

> Replacement for msg.sender. Returns the actual sender of a transaction: msg.sender for regular transactions, and the end-user for GSN relayed calls

{% hint style="info" %}
If `a` wants to send 10 ERC-20 tokens to `b`, then `a` calls `transfer(b, 10)`. If `b` now checks his balance (e.g. because `a` informs him that he has just sent him 10 tokens), he will find that it has increased by 10 tokens. If your friends want to send tokens to each other, `transfer` is the way.
{% endhint %}
