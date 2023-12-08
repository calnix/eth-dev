---
description: Based on LooksRare
---

# EIP-721

**EIP-712 aims to improve off-chain message signing**

* Currently, signed messages are an opaque hex string displayed to the user with little context about the items that make up the message.&#x20;
* EIP-712 aims to make these hex strings human readable.
* An EIP-712 signature allows signers to see exactly what they are signing in a client wallet as the signed data is split into different fields.
* It also prevents the reuse of signature - achieved by having a **domain separator** in the signature.

### domain seperator

**The domain separator prevents collision of otherwise identical structures.**

The domainSeparator is a value unique to each domain that is ‘mixed in’ the signature. This helps to prevent a signature meant for one dApp from working in another.

* The sellOrder hash contains all the necessary properties to be processed as a valid order, except the signature values.&#x20;
* EIP-712’s standard encoding prefix is \x19\x01, so the final digest is: `bytes32 digest = keccak256(abi.encodePacked("\x19\x01", domainSeparator, hash));`&#x20;
* The signature created by the seller is stored off-chain (v, r, s are the values for the transaction's signature).

<figure><img src="../../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

* Subsequently, when a prospective buyer wishes to make a purchase, he would do so with the buy function.&#x20;
* The buy function checks that the sellOrder struct submitted by the buyer lines up with the seller's signature that was previously stored off-chain.&#x20;
* ecrecover is used to verify the signature. ecrecover is simply recovering the public key (and from there the address) used to sign the digest.

### **On signing and verifying using ECDSA:**

* ECDSA signatures consist of two numbers (integers): r and s. Ethereum uses an additional v (recovery identifier) variable.
* r and s (ECDSA signature outputs), v (recovery identifier)

### On why v is either 27 or 28:

v is important because since we are working with elliptic curves, multiple points on the curve can be calculated from r and s alone. (two points of intersection) This would result in two different public keys (thus addresses) that can be recovered. The v simply indicates which one of these points to use.

* To create a signature you need the message to sign and the private key to sign it with
* The {r, s, v} signature can be combined into one 65-byte-long sequence: 32 bytes for r, 32 bytes for s, and one byte for v
* In order to verify a message, we need the original message, the address of the private key it was signed with, and the signature {r, s, v} itself.

## LooksRare

{% hint style="info" %}
&#x20;LooksRare uses exclusively [**EIP-712 signatures**](https://eips.ethereum.org/EIPS/eip-712)**,** for off-chain maker orders
{% endhint %}

* A maker order is a passive order, which can be executed (if not cancelled!) against a taker order after it is signed.&#x20;
* Since the execution of a maker order is done once a taker order matches on-chain, network gas fees are never paid by the maker user.
* A seller has to sign an EIP-712 signature of the order’s hash, which can then be later submitted on chain by the buyer.

