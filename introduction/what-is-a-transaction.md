# What is a Transaction

## Ethereum

* Each node can contain a full copy of the Blockchain.
* Anyone can run a node.
* Each node is a machine running an Ethereum client.
* Networks are formed by multiple nodes.
* There are many different Ethereum networks. (Ropsen, etc)
* Ethereum networks are used to transfer money and store data

### Transaction

<figure><img src="../.gitbook/assets/image (204).png" alt=""><figcaption></figcaption></figure>

* nonce -> (index) if this is the 101st transaction sent by sender, transaction will have nonce of 101.&#x20;
  * to avoid prevent replay attacks.
* Transactions are signed with the private key, generating v,r,s components.

### **Accounts and Signing**

**How does the blockchain know that the transaction sent is not forged?**

**Transactions are signed and can be verified to be authentic, like so:**

For a transaction signature to be created, access to the account and its private key is required. Which is why, we need to unlock our MM, and sign before committing a transaction.

**Private Key: 32 bytes (64 hex characters)**

<figure><img src="../.gitbook/assets/image (195).png" alt=""><figcaption></figcaption></figure>

**Public Key**

The public key is generated from a private key, using ECDSA. This is a one-way function, the only way is to brute-force it.

<figure><img src="../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

**Ethereum Account (Wallet Address)**

An Ethereum account is the Keccak Hash of the last 20 bytes (40 hex char) of the public key.

<figure><img src="../.gitbook/assets/image (140).png" alt=""><figcaption></figcaption></figure>

> _**Can go from Private -> Public -> Account. No reverse.**_

**Signing & Verification**

The transaction gets signed with the private key, creating additional fields of v,r,s. The signature is the r and s.

_You cannot reverse engineer the private key from the signature._

<figure><img src="../.gitbook/assets/image (185).png" alt=""><figcaption></figcaption></figure>

With the r & s, you can run them through an ECRECOVER function, and it will output the Public Key and Wallet Address (Eth account).

**This authenticates the transaction.**

**What is the difference between EOA and Contract Accounts?**

* Externally-owned account (EOA) – controlled by anyone with the private keys, can initiate transactions
* Contract account – a smart contract deployed to the network, controlled by code initiated from an external EOA or another contract

**What does an Ethereum Account contain?**

* `nonce` – a counter that increments with every transaction sent from the account. This ensures transactions are only processed once. For contract accounts, it the number of contracts created by the account.
* `balance` – amount of wei owned by the address
* `codeHash` – the hash of the code of an account on the EVM. Contract accounts contain code fragments that can perform different operations. This code gets executed if the account receives a message call. It cannot be changed, unlike the other account fields. For EOAs, the codeHash field is the hash of an empty string.
* `storageRoot` – keccak256 hash of the root node of a Merkle Patricia trie that encodes the storage contents of the account. empty by default.

**What information does a transaction contain?**

A submitted transaction includes the following information:

* `recipient` – the receiving address (if an EOA, the transaction will transfer value. If a contract account, the transaction will execute the contract code)
* `signature` – the identifier of the sender. This is generated when the sender's private key signs the transaction and confirms the sender has authorized this transaction
* `nonce` - a sequencially incrementing counter which indicates the transaction number from the account
* `value` – amount of ETH (in wei) to transfer from sender to recipient
* `data` – optional field to include arbitrary data
* `gasLimit` – the maximum amount of gas units that can be consumed by the transaction
* `maxPriorityFeePerGas` - the maximum amount of gas to be included as a tip to the validator
* `maxFeePerGas` - the maximum amount of gas willing to be paid for the transaction (inclusive of `baseFeePerGas` and `maxPriorityFeePerGas`)

**What is the life cycle of a transaction from when the user signs to when it is included in the blockchain?**

1. Once you send a transaction, cryptography generates a transaction hash from the signed transaction (transaction fields and v,r,s)
2. The transaction is then broadcast to the network and included in a pool with lots of other transactions.
3. A validator must pick your transaction and include it in a block in order to verify the transaction and consider it "successful".
4. As time passes the block containing your transaction will be upgraded to "justified" then "finalized". Protocol upgrades make it much more certain that your transaction was successful and will never be altered. Once a block is "finalized" it could only ever be changed by an attack that would cost many billions of dollars.

**How does the current Proof of Stake consensus mechanism in Ethereum work?**

* Validating nodes have to stake 32 ETH into a deposit contract as collateral against bad behavior. This helps protect the network because dishonest activity leads to partial or full slashing.
* In every slot (spaced twelve seconds apart) a validator is randomly selected to be the block proposer. They bundle transactions together, execute them and determine a new 'state'. They wrap this information into a block and pass it around to other validators.
* Other validators who hear about a new block re-execute the transactions to ensure they agree with the proposed change to the global state. Assuming the block is valid they add it to their own database.
* If a validator hears about two conflicting blocks for the same slot, they will pick the one supported by the most staked ETH.
* When the network performs optimally and honestly, there is only ever one new block at the head of the chain, and all validators attest to it. However, it is possible for validators to have different views of the head of the chain due to network latency or because a block proposer has equivocated.
* Therefore, consensus clients require an algorithm to decide which one to favor. The algorithm used in proof-of-stake Ethereum is called [LMD-GHOST](https://arxiv.org/pdf/2003.03052.pdf), and it works by identifying the fork that has the greatest weight of attestations in its history.

