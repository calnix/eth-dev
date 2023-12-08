# Events

## Events

* Events allow us to “print” information on the blockchain in a way that is more searchable and gas efficient than just saving to storage variables
* Events are not stored in a contract like regular storage, but rather in a special logs section of a transaction.
* Logs are gas efficient way to check for specific data that's not required for the contract itself. **Cannot be accessed by smart contracts**
* Applications (outside of blockchain) can subscribe and listen to these events through the RPC interface of an event client.

{% hint style="info" %}
To put it simply, **events** are ways to communicate with a client application or front-end website that something has happened on the blockchain.
{% endhint %}

Event arguments marked as **indexed, can be searched**&#x20;

```solidity
// Format:
// can take up to 3 indexed parameters
event MyEvent(uint _arg1, address indexed _arg2)
```

* Up to 3 parameters can be indexed; these are referred to as topics.
* if you declare an argument to be `indexed`  it will be hashed, and can look them up
* Logs and their events data **cannot be retrievable** from within contracts.
* Events are **inheritable** members of a contract.
* Events are **cheap**.

#### Example

```solidity
// emit event when name is registered or relinquished
event NameRegistered(address indexed owner, string indexed name);
event NameRelinquished(address indexed owner, string indexed name);

// register an available name
function registerName(string memory _name) public {
    require(nameOwner[_name] == address(0), "Already registered");
    nameOwner[_name] = msg.sender;
    emit NameRegistered(msg.sender, _name);
}
```

### Etherscan

![](<../../.gitbook/assets/image (346).png>)

* **Address**: address of contract that emitted the event
* **Topics**: indexed parameters of events
* **Data**: abi-encoded non-indexed parameters of events
  * have to decode using the abi of the contract
  * if contract is verified on etherscan, can view in decoded mode 'dec'

![https://kovan.etherscan.io/address/0x136607f1f09312d041b2e60396105f58bd063d66#events](<../../.gitbook/assets/image (87).png>)

**topic\[0]** always refers to the hash of the event itself `keccak256(NameRegistered(<address>, <name>))`

{% hint style="danger" %}
**Indexing strings:**\
\
Fundamentally, it can be done.

* Indexed event arguments are stored in topics, and must be of fixed size;&#x20;
* Solidity hashes the string and stored it in the topic. Filtering by the hash value of string would work.
* However, hash functions are one-way. We cannot decode the hashes to obtain the string.
* [https://ethereum.stackexchange.com/questions/6840/indexed-event-with-string-not-getting-logged](https://ethereum.stackexchange.com/questions/6840/indexed-event-with-string-not-getting-logged)
{% endhint %}

### **Links**

* https://blog.chain.link/events-and-logging-in-solidity/&#x20;
* https://www.youtube.com/watch?v=w18c9HLEuBs
