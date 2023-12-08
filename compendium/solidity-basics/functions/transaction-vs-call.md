# Transaction vs Call

There are two ways we can invoke functions.

1. Call
2. Send a transaction (data modification)

![](<../../../.gitbook/assets/image (71).png>)

{% hint style="info" %}
A transaction is necessary, if a value in a Smart Contract needs to be updated (state change).&#x20;

A call is done, if a value is read.&#x20;

Transactions cost Ether (gas), need to be mined and therefore take a while until the value is reflected, which you will see later. Calls are virtually free and instant.
{% endhint %}

![transaction followed by call](<../../../.gitbook/assets/image (214).png>)

### Sending a transaction to a function

Anytime we send a transaction to the network it takes time to be process (it must be confirmed by other nodes and mined).

When we send a transaction to a function, we are returned a transaction hash that identifies the transaction.&#x20;

* We do not get a return value back, even if the function has a return component in it.
* Costs money.

### Transactions

Transactions always start with an externally owned account signing a transaction and submitting it to the network. The returned value is a transaction receipt.&#x20;

The outcome of the transaction is _**unknown**_ because nothing has happened yet except that a _**request**_ has been submitted to the network for verification _**in the future**_.

A typical approach to coping with this fact of life is to wait for the transaction to be mined, then inspect the transaction log for useful output, inspect the success/fail status and draw your own conclusions, or inspect a read-only function to discover the new state after the transaction was mined.
