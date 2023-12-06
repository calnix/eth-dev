# Merkle Trie

**In blockchain:**

* Data structure made up of hashes of various data blocks that summarize all the transactions in a block.
* Used to encrypt blockchain data more efficiently and securely.
* Enables quick and secure content verification across big datasets and verifies the consistency and content of the data.

**In solidity:**

Merkle trees are a data structure that allow efficient and secure verification of the integrity of data. They are commonly used in Solidity to efficiently verify large amounts of data with minimal on-chain storage (gas cost).

Merkle trees provide several benefits for Solidity developers:

1. **Efficient verification** of large datasets can be particularly useful for blockchain applications where data is often too large to store on-chain
2. **Reduced gas costs** due to less data being stored on-chain
3. **Increased security** Merkle trees provider a reliable way of verifying data integrity

{% hint style="info" %}
Merkle tree allows you to cryptographically prove that an element is contained

in a set **without revealing the entire set.**
{% endhint %}

### Structure

At the root of the **Merkle tree** is the root hash. It's created by hashing all its original values as leaf nodes. Now two leaf hashes are combined by creating a new hash just for those together. We do this all the way until we have one tree with a single root hash.

Just hash pairwise from bottom to top, till you get to the root.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* Merkle trees are made by hashing pairs of nodes repeatedly until only one hash remains; this hash is known as the Merkle Root or the Root Hash.
* They're built from the bottom, using Transaction IDs, which are hashes of individual transactions.&#x20;
* Each non-leaf node is a hash of its previous hash, and every leaf node is a hash of transactional data.

### Uses

* easy to verify data integrity: one value changes, the entire root hash changes
* easy to **verify inclusion of a specific transaction** (merkle proof)

#### Why not just concatenate the entire transaction series and hash? why tree?

* must harder to verify if a specific transaction is included in the concat method.
* you would need all the other transactions. problematic for larger data sets.

Hence, trie solves this and therefore the second property of **verify inclusion of a specific transaction.** This makes light client possible - no longer need the record of all transactions, just need to hold block headers, which include the merkle root.&#x20;



## Merkle Patricia Trie

Block header in Ethereum contains 3 merkle trees:

* Transactions
* Receipts
* State

#### State Tree

* key-value mapping&#x20;
* keys are account addresses,
* values are account declarations: balances, nonce, code (if contract account)

#### Transaction Tree

* key-value mapping where key is the transaction ID

#### Why MPT instead of regular merkle tree for Ethereum?

* Making updates in different orders should result in the same state and tree root
* In a regular merkle tree, the root hash value depends on the order of leaves; sensitive or order.

### What is a merkle patricia trie?

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption><p>Inefficient</p></figcaption></figure>

\
