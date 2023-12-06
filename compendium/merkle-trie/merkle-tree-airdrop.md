# Merkle Tree Airdrop

1. Create a whitelist of addresses as a merkle tree off-chain
2. Store the root of t he merkle tree on-chain
3. On-chain verification&#x20;

#### Why not just make a list, concatenate and hash it. Then store in the contract?

* It would be a fingerprint of the data
* But we cannot produce merkle proofs from it
* If you want to **verify the inclusion of a specific data (address)** you would need to provide your data, and all the other data to produce the final hash.
* This is why we need the tree structure. With the tree structure, you can approve inclusion in the original set of data efficiently.
* &#x20;

### How to check inclusion

You will need to **provide the data of the leaf** you want to check. And **all the hashes to get it from that leaf to the root**.

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

* Let's assume that we want to check inclusion of the red leaf.&#x20;
* We will need to provide that specific's leaf's data, as well as all the hashes of the blue leaves.
* With the first blue leaf hash and the provided data, we can hash the first brown node above.
* Now, we will need the hash of the second blue leaf, to hash against it, and so forth until we reach the root.
* Upon reaching the root, we will check if this generated root matches the root stored on-chain. If so, inclusion is confirmed.

### Complexity

Each proof of inclusion, requires 1 additional hash per level of the tree. In the example above, we had to ascend 3 levels, and needed 3 other hashes.&#x20;

**Height of tree is logarithmic in the number of data leaves.** Meaning, the number of leaves could be huge, but the height of the tree would not be very tall.

For example, if you have a million leaves, the height of the tree would be around 20:

```
// million leaves -> 20 levels
2^20 = 1,048,576
2^21 = 2,097,152

// billions leaves -> 30 levels
2^30 = 1,073,741,824
2^31 = 2,147,483,648
```

As the number of leaves increases massively, the height doesn't increase that much. Take a large number, how many times do you have to divide it by 2, before getting to 1. It's not as much as you think.

Complexity = O(log2(n))  \[log base 2, where n is the number of leaves]

{% hint style="info" %}
Therefore,  producing and checking the proof is very efficient. Contract only stores the root with 256 bits.
{% endhint %}

## Why this works

For any given leaf that we want to verify, there is effectively a single set of intermediate hashes that can produce the final root hash. No other hashes will work.

So while the contract does not "know" what those intermediate hash values are, it can identify fakes because they won't produce the final root hash, which the contract checks against.

### Bitmaps

So far we don't have a way to prevent replay attack - users from repeatedly claiming their airdrop.&#x20;

We need a way to record if a user has claimed their airdrop.&#x20;

Other ways that are inefficient

1. mapping (address => bool):&#x20;

* an entire storage slot will be taken for **each boolean.** There is no packing. &#x20;
* For mapping values there is no way to order them to save space by fitting smaller types into a single slot like with arrays.&#x20;
* Cos of the way you calculate the storage location of each value from the key and index slot.

{% hint style="info" %}
A bool is a uint8 under the hood. And all uints are basically uint256, in that they take up 1 byte
{% endhint %}

2. array bool\[]&#x20;

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

That's a lot of bits for just a true/false. Wasting storage space.&#x20;

#### Bitmap implementation

We can use a dynamic array **uint256\[]**

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

* The first uint256 in the array will hold the bools for the first 256 addresses.&#x20;
* We will need to index across and within the uints. First we need to find which uint the bit exists within.&#x20;
* We index our addresses off-chain. Let's say we want to lookup the address with an index of 525.
* 525 // 256 = 2  (ignore the decimals). so we want the 3rd uint in the array
* Which bit in this specific&#x20;
* 525 % 256  = 13
* 13th bit in the 3rd uint

OZ instead does, a **mapping(uint256 => uint256)**&#x20;

* storage slot in a dynamic array keeps track of the length
* but for a mapping, while a storage slot is set aside, nothing is stored within in.&#x20;
* this makes it slightly cheaper.

### Code

* [https://github.com/jordanmmck/MerkleTreesBitmaps](https://github.com/jordanmmck/MerkleTreesBitmaps)

To verify the proof, the contract will require the intermediate hashes along with the leaf data for verification.&#x20;

### Website/backend

* When a whitelisted user attempts to claim their tokens. All that really needs to be done is to implement some JavaScript, on the website that makes a fetch request to an external API while on the mint page.&#x20;
* This API will receive the connected wallets address, as this is what we originally used to generate our leaf nodes, and return the designated proof.
* Server-side, you would receive the address, hash it using keccak256, and retrieve the proof&#x20;

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>
