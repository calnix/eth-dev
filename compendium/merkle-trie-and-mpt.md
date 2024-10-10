# Merkle Trie and MPT

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

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

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

## Patricia Merkle Trees

An ordered tree data structure that is used to store a dynamic set or associative array where the keys are usually strings. A node's position in the tree defines the key with which it is associated.

\


<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption><p>Inefficient</p></figcaption></figure>

### Ethereum

Ethereum uses **three** trie structures. You might be wondering why multiple tries. Well, it is that we have to deal with permanent data like transactions in a committed block as well as temporary data like account balance which gets updated. Data structures are demanded to store them separately.

Block header in Ethereum contains 3 merkle trees roots:

* Transactions (transactionRoot)
* Receipts (receiptsRoot)
* State  (stateRoot)

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

**State trie:**&#x20;

* The state trie / world state trie represents a mapping between account addresses and the account states. The account state includes the _balance, nonce, codeHash, and storageRoot._&#x20;
* Here the _key_ is a 160 bit address of an Ethereum account and the _value_ is the account state.&#x20;
* The storageRoot itself is the root of an account storage trie, which stores the contract data associated with an account.&#x20;
* The root node depends on all internal data and its hash value acts as a unique identifier for the entire system state.

Ethereum accounts are added to the state trie only after a successful transaction is recorded against that account. The world state trie stores some temporary data (like balances) which gets updated, changing the root hash frequently. The stateRoot is hash of the root node of the state trie, after every transactions are executed and committed.

**Transaction trie:**&#x20;

* It is created on the list of transactions within a block. The path to a specific transaction in the transaction trie is tracked based on the position of the transaction within the block.&#x20;
* Once mined, the position of a transaction in a block does not change. So a transaction trie never gets updated.
* This is similar to the Merkle tree representation of transactions in Bitcoin and the transaction verification can be done similarly. The transactionRoot is the hash of the root node of the transaction trie.

**Receipt trie:**&#x20;

* The receipt records the result of the transaction which is executed successfully. It consists of four items: _status code of the transaction, cumulative gas used, transaction logs,_ [_Bloom filter_ ](https://ethereum-classic-guide.readthedocs.io/en/latest/docs/appendices/bloom\_filters.html)_(a data structure to quickly find logs)._&#x20;
* Here the key is an index of transactions in the block and the value is the transaction receipt. Like transaction trie, receipt trie never gets updated.&#x20;
* The receiptsRoot is the hash of the root node of the trie structure populated with the receipts of each transaction in the transactions list portion of the block;

**Example of storing the mapping between account address and ether balance.**

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>



### Merkle Vs Merkle-Patricia <a href="#id-784e" id="id-784e"></a>

Since we have seen both Merkle tree and Merkle Patricia tree, let us analyze some of the major differences that keeps them separate.

1\. Root of a Merkle Patricia trie does not depend on the order of data. However, Merkle root depends on order of data, since we are pairing adjacent nodes and taking the hash value.

2\. Tree size of Merkle Patricia trie is lower than a standard Merkle tree due to prefix based compression techniques.

3\. Merkle patricia tries are faster than Merkle trees, but the implementation is complicated.

\
