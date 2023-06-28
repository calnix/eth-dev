# Return & Events

## Returns

* Functions can have an optional return statement; not restricted to pure & view only.&#x20;
* If function is not (pure,view,constant) -> returned-value are restricted to on-chain usage.
* If function is (pure,view,constant) -> returned-values can be off-chain.

### Functions without `pure`, `view`, `constant` modifiers

Returned values **can only be fetched by on-chain calls** (contract functions), but not by off-chain calls (Externally Owned Address/wallets).

When you call such function from off-chain you are **returned the transaction hash.**

* This is because it is unknown when the transaction will be mined and added to the blockchain.
* Only when it is successfully mined and computed, do we reach the actual value of the returns statement in the function.&#x20;
* This value from the returns statement can be used **only inside the blockchain** (contract functions).
* If you want this value to be accessible off-chain, you need to generate an `event` (inside the Solidity function) to emit it to listeners.

{% hint style="warning" %}
_no way for a transaction from an EOA to return a value that can be read, even by examining the blockchain._
{% endhint %}

### Functions that are `pure`, `view`, `constant`&#x20;

* These functions do not alter states or interact with the blockchain. As such, there is no transaction hash involved, as nothing is submitted for mining.&#x20;
* These "read-only" functions can return values for off-chain usage.&#x20;

### Example

If you want to `return true`, as below, the user interacting with it will not receive it as output. To achieve this, use events.

{% tabs %}
{% tab title="Return" %}
```solidity
//SPDX-License-Identifier: MIT
pragma solidity 0.8.3;

contract EventExampel {
mapping(address => uint) public tokenBalance;

constructor(){
    tokenBalance[msg.sender] = 100;
    }

function sendToken(address _to, uint _amount) public returns(bool){
    require(tokenBalance[msg.sender] >= _amount, "Not enough tokens");
    tokenBalance[msg.sender] -= _amount;
    tokenBalance[_to] += _amount;

    return true;
    }
}
```
{% endtab %}

{% tab title="Events" %}
```solidity
//SPDX-License-Identifier: MIT
pragma solidity 0.8.3;
contract EventExampel {

mapping(address => uint) public tokenBalance;
event TokenSent(address _from, address _to, uint _amount);

constructor(){
    tokenBalance[msg.sender] = 100;
}

function sendToken(address _to, uint _amount) public returns(bool){
    require(tokenBalance[msg.sender] >= _amount, "Not enough tokens");
    tokenBalance[msg.sender] -= _amount;
    tokenBalance[_to] += _amount;

    emit TokenSent(msg.sender, _to, _amount);
    return true;
    }
}
```
{% endtab %}
{% endtabs %}

* Create event `TokenSent`, taking in parameters that we want to emit to the outside world.&#x20;
* Then we emit the event before we are returning from the function.&#x20;

## Events

Events allow us to “print” information on the blockchain in a way that is more searchable and gas efficient than just saving to public storage variables in our smart contracts.

Events are not stored in a contract like regular storage, but rather in a special logs section of a transaction.

Logs are gas efficient way in Ethereum to check for specific data that's not required for the contract itself. **Cannot be accessed by smart contracts**

Applications (outside of blockchain) can subscribe and listen to these events through the RPC interface of an event client.

```solidity
// Format:
// can take up to 3 indexed parameters
event MyEvent(uint _arg1, address indexed _arg2)
```

* Event arguments marked as **indexed can be searched** for
  * Up to 3 parameters can be indexed.
  * if you declare an argument to be `indexed` -> it will be hashed, and can search for them on the side-chain.
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
        require(nameOwner[_name] == address(0), "The provided name has already been registered!");
        nameOwner[_name] = msg.sender;
        emit NameRegistered(msg.sender, _name);
    }
```

![https://kovan.etherscan.io/address/0x136607f1f09312d041b2e60396105f58bd063d66#events](<../.gitbook/assets/image (206).png>)

* `topic[0]` always refers to the hash of the hash of the event itself keccak-256(NameRegistered(\<address>, \<name>))

{% hint style="danger" %}
**Indexing strings:**\
\
Fundamentally, it can be done.

Indexed event arguments are stored in topics, and must be of fixed size; Solidity hashes the string and stored it in the topic. Filtering by the hash value of string would work.

However, hash functions are one-way. We cannot decode the hashes of a specific address to get its list of names. Hence, I decided to not index string in my revised code.

Reference:\
[https://ethereum.stackexchange.com/questions/6840/indexed-event-with-string-not-getting-logged](https://ethereum.stackexchange.com/questions/6840/indexed-event-with-string-not-getting-logged)
{% endhint %}

{% hint style="info" %}
Further reading:

[https://blog.chain.link/events-and-logging-in-solidity/#:\~:text=One%20such%20important%20piece%20of,data%20structure%20on%20the%20blockchain.](https://blog.chain.link/events-and-logging-in-solidity/)\
\
[https://soliditydeveloper.com/solidity-fast-track-2](https://soliditydeveloper.com/solidity-fast-track-2)
{% endhint %}

### Use cases

Since `returns` in transactions cannot return values.&#x20;

* Used for return values from transactions
* Used externally to trigger functionality in other SC
* Used as a cheap data storage

### Events as Return Values

* Writing Functions cannot return data externally
* Instead they return a transaction hash
* Transactions can take long or may fail

Events are used to inform the external user that something has happened, since writing functions cannot return data externally.&#x20;

### Events as Trigger

![](<../.gitbook/assets/image (135).png>)

When Alice withdraws 3 ETH from the 2/2 Multi-Sig wallet, an event will be emitted to inform Bob of this.

Bob would be using an external application listening for these events to alert him to approve the request accordingly.&#x20;

### Events as Data storage

Problem: Storing data on Ethereum is extremely expensive.

Solutions:

* Store data off-chain, and store only the proof(hash) on the chain.
* Store data on another blockchain like IPFS
* Store data in Event logs if not necessary.
  * If data is not needed directly by the smart contract put it in a log

#### Further reading:

[https://blog.chain.link/events-and-logging-in-solidity/#:\~:text=One%20such%20important%20piece%20of,data%20structure%20on%20the%20blockchain.](https://blog.chain.link/events-and-logging-in-solidity/)

[https://soliditydeveloper.com/solidity-fast-track-2/](https://soliditydeveloper.com/solidity-fast-track-2/)
