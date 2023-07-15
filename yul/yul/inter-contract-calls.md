---
description: >-
  https://github.com/RareSkills/Udemy-Yul-Code/blob/main/Video-12-13-External-Calls.sol
---

# Inter-contract calls

The hex data field in a transaction is tx.data, or msg.data

* when sending to a wallet, you don't have to put any data, unless you are trying to send that person a message (try hackers and taunts1)
* you can send a transaction with no value, just a message in the hex field

#### When sending to a contract, the first four bytes specify which function you are calling, and the bytes that follow are abi encoded function parameters.&#x20;

* solidity expects the bytes **after the function selector** to always be a multiple of 32 in length, but this is convention.
* if you send more bytes, solidity will ignore them
* But a Yul contract can be programmed to respond to arbitrary length tx.data in an arbitrary manner

**Example of txn data:**

<figure><img src="../../.gitbook/assets/image (60) (1).png" alt=""><figcaption></figcaption></figure>

* 1st four bytes are fn selector,&#x20;
* concatenated to it is the first argument
* addresses have 20 bytes, but abi encoded data is always a multiple of 32 bytes, the **address is left padded with zeros**&#x20;

<figure><img src="../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

* first 4 bytes is fn selector
* next 32 bytes is the first argument: left padded with zeros, since address takes 20 bytes
* next 32 bytes is the 2nd argument: left padded with zeros, with inte

{% hint style="info" %}
It doesn't matter if the fn argument was a `uint256` or `uint128`, etc, the abi encoded data  will always be 32 bytes
{% endhint %}

<figure><img src="../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

## Staticcall

<figure><img src="../../.gitbook/assets/image (228).png" alt=""><figcaption></figcaption></figure>

**Parameters for staticcall:**

* how much gas to send -> `gas()`
  * means we are passing all remaining gas to it.&#x20;
  * can hardcode for a specific smaller amount. useful if you worry about DoS&#x20;
* address of contract we are calling:  `_a`
* intended txn.data: region in memory -> 28, 32&#x20;
* region in memory to copy results into: `0x00` to `0x20`
  * when the fn returns, it will overwrite the area containing the address param, but that's fine, since we don't need it anymore

## Dynamic Length Arguments

<figure><img src="../../.gitbook/assets/image (238).png" alt=""><figcaption></figcaption></figure>

If we pass, `a = 7`, `b = [1,2,3]`, `c = 9,` **this is what the calldata will look like:**

<figure><img src="../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

* 0x00 -> 7&#x20;
* 0x20 -> pointer to where the array begins, which is 0x60 - 0xc0
* 0x40 -> 9
* 0x60 - 0xc0 -> \[1,2,3]
