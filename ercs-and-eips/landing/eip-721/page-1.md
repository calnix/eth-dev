# Page 1

<figure><img src="../../../.gitbook/assets/image (233).png" alt=""><figcaption></figcaption></figure>

* Seller creates a sell order -> sellOrder struct with relevant info (price, NFT addr)
* sellOrder struct is hashed to get sellOrderDigest
* The seller then signs an EIP-712 signature of the sell order’s hash, which is stored off-chain.
  * domainSeparator is a value unique to each domain that is ‘mixed in’ the signature - prevents collision of otherwise identical structures.
* EIP-712’s standard encoding prefix is \x19\x01, so the final digest is \
  `bytes32 digest = keccak256(abi.encodePacked("\x19\x01", domainSeparator, hash));`
  * ```
    (v, r, s are the values for the transaction's signature)
    ```

#### sellOrderDigest

<figure><img src="../../../.gitbook/assets/image (213).png" alt=""><figcaption></figcaption></figure>

* sellOrderDigest is the keccak256 hash of all the MakerOrder properties set when creating an order and a constant value.
* The constant value is the typehash

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

* `SELL_ORDER_TYPEHASH` -> keccak256 hash of the SellOrder struct and its fields
* `typeHash` is to separate types -> collision resistance
  * By giving different types a different prefix the hash function only has to be injective within a given type.&#x20;
  * It is okay for `hash(a)` to equal `hash(b)` as long as `typeOf(a)` is not `typeOf(b)`.

{% hint style="info" %}
Every hash function is NOT injective. Hashes map a large domain to a significantly smaller codomain. By the pigeon-hole principle, such a function can't be injective, because there would be items in the domain that'll map to the same item in the codomain.
{% endhint %}

### Buying

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (226).png" alt=""><figcaption></figcaption></figure>

* calculate the EIP721 digest of the sellOrder inputted
* use ecrecover together with this digest and the v,r,s values to generate the signer address
* check if this signer address generated matchs the one within the sellOrder.signer

### ecrecover

* _recover_ the address of the private key that a message was signed with.&#x20;
* whenever you send a transaction to the Ethereum network, you have to sign this transaction with your private key. Ethereum nodes have some way to verify a signature.
* This functionality of verifying a signature was added to the smart contract itself.&#x20;
* With this you can verify much more than just the transaction signature itself. In fact you can pass any data to a smart contract, hash it and then verify its signature against the data. The signature is the combination of v, r and s.

### On signing and verifying using ECDSA

* ECDSA signatures consist of two numbers (integers): r and s. \
  Ethereum uses an additional v (recovery identifier) variable.
* ```solidity
  r and s (ECDSA signature outputs), v (recovery identifier)
  ```

{% hint style="info" %}
r, s and v are obtained from the signature of the transaction.\


```ini
var sig = secp256k1.sign(msgHash, privateKey)
r = sig.signature.slice(0, 32)
s = sig.signature.slice(32, 64)
v = sig.recovery + 27
```
{% endhint %}

#### v: recovery identifier

any `r` can generate two valid signatures, and a few of them can generate up to 4 (due to it being an elliptic curve). Each of them leads to the recovery of a different public key, and so we need a way to distinguish which is the real one.

This is why Ethereum signatures include a third element, the value `v`, called the _recovery identifier_

\


### Security issues with ecrecover + solution

1. In some cases ecrecover can return a random address instead of 0 for an invalid signature. This is prevented above by the owner address inside the typed data.
2. Signature are [malleable](http://coders-errand.com/malleability-ecdsa-signatures/), meaning you might be able to create a second also valid signature for the same data. In our case we are not using the signature data itself (which one may do as an id for example).
3. An attacker can construct a hash and signature that look valid if the hash is not computed within the contract itself.

\
