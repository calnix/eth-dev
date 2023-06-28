# TLDR transfer vs transferFrom

* Use **transfer** when when we want to initiate a transfer of tokens **from ourselves (us as the function caller) to another address.**
* Use transferFrom when **an external entity is requesting a transfer of tokens from some party A to itself**. This is require allowance for the external entity to be set by part A
  * hence, \_spendAllowance will decrement allowance
  * then, \_transfer moves the tokens from A to external entity.

{% hint style="info" %}
**transferFrom()** allows a third-party (who has authorization to spend on your behalf, which is called with `approve()` function) to send tokens.&#x20;

This opens up some things;&#x20;

1. Someone paying for your gas fees to transfer tokens.
2. Someone being able to transfer your tokens (think direct debit).
3. Contracts, with authorization, can spend your tokens.



You've probably encountered this function call (and the two step UIUX), that you need to press "Approve"/"Enable" buttons on dApps for token interactions - this is what it is doing.
{% endhint %}
