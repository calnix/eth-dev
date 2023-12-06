---
description: https://www.rareskills.io/post/merkle-tree-second-preimage-attack
---

# Merkle Tree: shortened proof attack



The _second preimage attack_ in Merkle trees can happen when an intermediate node in a merkle tree is presented as a leaf.

### Valid example

<figure><img src="../.gitbook/assets/image (348).png" alt=""><figcaption></figcaption></figure>

* we want to create a merkle proof for the leaf ℓ₂, in yellow text.
* we will need all the hashes in green.

The proof to the root is

```bash
h(a) = h(h(ℓ₁) + h(ℓ₂))
h(e) = h(h(a) + h(b))
h(g) = h(h(e) + h(f))
return root == h(g)
```

### Attack

What if the attacker provides an intermediate node as a leaf, along with the shortened proof?

Essentially, if a merkle proof is valid, then a shortened version of it is also valid if we pass the first value in the original proof as a leaf.

<figure><img src="../.gitbook/assets/image (349).png" alt=""><figcaption></figcaption></figure>

* attacker provides a as the leaf, and the green hashes as the merkle proof
* the merkle can be correctly generated from this truncated offering, and therefore verification will pass

### Prevention

#### 1. The attack requires 64 byte leaves <a href="#viewer-50dao" id="viewer-50dao"></a>

For this attack to work, the attacker must pass in the preimage of the intermediate node, not its hash value. This means it must pass **a** as the leaf, not **hash(a)**. Since Solidity uses 32 byte hashes, **a = h(ℓ₁) + h(ℓ₂)** will be 64 bytes long.

* If the contract does not accept 64 byte leaves as input, then the attack will not work.
* That is, if the input to the leaf has a length other than 64 bytes, it is impossible to pass **h(ℓ₁) + h(ℓ₂)** as a false leaf value.

#### 2. Using a different hash for the leaf as a defense <a href="#viewer-9ol37" id="viewer-9ol37"></a>

If our application need to accept 64 byte leaves for some reason, then we can prevent the attack by using a different hash for the leaves than for the proof.

* That is, when the leaf is first hashed, we used a different hash than what we use for hashing to the root.&#x20;
* This will prevent the attacker from “reconstructing” an intermediate node as if it were a leaf. Using the diagram above, the attacker is using **h(a)** to create the false “leaf.” However, if leaves are passed through **h’(a)**, then the intermediate value cannot be reconstructed.

**OpenZeppelin uses a double hash as a "different hash"**

Instead of using a different hash function (such as via a [precompile](https://www.rareskills.io/post/solidity-precompiles)) which would cost more gas, Openzeppelin simply hashes the leaf twice. That is, **h’(x) = h(h(x))**.

We've used green underlines to show where the hash is taken twice to construct the leaf node from the underlying data.

<figure><img src="../.gitbook/assets/image (351).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
The attack is easy to defend against — simply do not allow non-leaf values to be interpreted as leafs.
{% endhint %}
