# Introduction

## What is a smart contract?

A smart contract is **an agreement between two people or entities in the form of computer code programmed to execute automatically**.

* Coined by computer scientist Nick Szabo in 1994
* Smart contracts are executed on blockchain, which means that the terms are stored in a distributed database and cannot be changed.&#x20;
* Transactions are also processed on the blockchain, which automates payments and counterparties. Since the emergence of the digital currency Ethereum, the creation and execution of smart contracts has been simplified, as complex transactions can be programmed into the Ethereum protocol.

## What is solidity?

Solidity is an object-oriented programming language created specifically for designing smart contracts on blockchain networks.

* .sol files
* Strongly Typed
* Similar to JavaScript in syntax

Contracts in Solidity are akin to classes in object-oriented languages. They include state variables that contain persistent data as well as functions that can manipulate the data in the state variables.

#### Example

```solidity
//define solidity version
pragma solidity ^0.8.0;

// define contract name: MyContract
contract MyContract { 

    // declare storage var
    string public message; 

    // declare functions
    function setMessage(string memory newMessage) public { 
        message = newMessage; 
    } 

    function getMessage() public view returns (string memory) { 
        return message; 
    } 
}
```

**Pragma: compiler versions**

`pragma solidity ^0.8.0;` -> specifies the version of Solidity that the code is written in.&#x20;

* You can choose the version to be an exact version like "0.8.4", or you could add the "`^`" to specify all versions above greater or equal to 0.8.0, which means, it will download the latest 0.8.x version.

<figure><img src="../.gitbook/assets/image (287).png" alt=""><figcaption></figcaption></figure>

**Bytecode**&#x20;

When we compile solidity code, it is translated into bytecode, something only the EVM can understand.

**ABI**&#x20;

* Key for applications to interact with the smart contract on the blockchain.&#x20;
* JavaScript applications, front-ends/back-ends, cannot directly interact with the bytecode deployed for the EVM.&#x20;
* The ABI serves as an interface/ layer of translation allowing for smart contract interaction.

{% hint style="info" %}
Similar to API
{% endhint %}

