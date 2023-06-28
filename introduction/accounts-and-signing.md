# Accounts and Signing

### How does the blockchain know that the transaction sent is not forged?

#### Transactions are signed and can be verified to be authentic, like so:

For a transaction signature to be created, access to the account and its private key is required. Which is why, we need to unlock our MM, and sign before committing a transaction.

#### **Private Key: 32 bytes (64 hex characters)**

![](<../.gitbook/assets/image (242).png>)

#### Public Key

The public key is generated from a private key, using ECDSA. This is a one-way function, the only way is to brute-force it.

![](<../.gitbook/assets/image (177).png>)

#### Ethereum Account (Wallet Address)

An Ethereum account is the Keccak Hash of the last 20 bytes (40 hex char) of the public key.

![](<../.gitbook/assets/image (294).png>)

{% hint style="danger" %}
Can go from Private -> Public -> Account. No reverse.
{% endhint %}

### Signing & Verification

The transaction gets signed with the private key, creating additional fields of v,r,s. The signature is the r and s.

_You cannot reverse engineer the private key from the signature._&#x20;

![](<../.gitbook/assets/image (314).png>)

With the r & s, you can run them through an ECRECOVER function, and it will output the Public Khey and Wallet Address (Eth account).

**This authenticates the transaction.**&#x20;
