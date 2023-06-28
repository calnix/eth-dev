---
description: >-
  https://jeancvllr.medium.com/solidity-tutorial-all-about-libraries-762e5a3692f9 
  https://medium.com/coinmonks/all-you-should-know-about-libraries-in-solidity-dd8bc953eae7
---

# Libraries

## TLDR

You can use a library in two ways

1. import, then call library functions: `myLibrary.someFunction();`
2. import then attach via using keyword: `using LibClone for address;`

In method two, we are attaching the functions in the library as methods to variable type as declared in the using statement.

**Example:**

```solidity
import {MathLib} from  './lib-file.sol';

contract test {
    using MathLib for uint;

    
    function testFn() external {
        uint a = 10;
        uint b = 10;
        uint c = a.subUint(b); // same as c = a - b
    }
}
```

* library functions are attached and can be applied to uint variables.
* uint variables have additional methods courtesy of the library.
* with this approach, when you call a library function, these functions will receive the object they are called on as their first parameter.
  * `subUint` takes 2 parameters, a and b.

{% hint style="info" %}
[https://jeancvllr.medium.com/solidity-tutorial-all-about-libraries-762e5a3692f9](https://jeancvllr.medium.com/solidity-tutorial-all-about-libraries-762e5a3692f9)\
[https://www.youtube.com/watch?v=25MLAnIzXRw](https://www.youtube.com/watch?v=25MLAnIzXRw)
{% endhint %}

## Old (for streamlining)

Similar to contract (`contract{}`), a library is a different type of Smart Contract that contains reusable code. Similar to a package in python.

Once deployed on the blockchain (only once) it is assigned a specific address, and its properties/methods can be used by other contracts in the Ethereum network.

They enable to develop in a more modular way. It is helpful to think of a library as a **singleton** in the EVM, a piece of code that can be called from any contract without the need to deploy it again.

```solidity
pragma solidity ^0.5.0;

library libraryName {
    // struct, enum or constant variable declaration
    // function definition with body
}

pragma solidity ^0.5.0;
library MathLib {
    
    struct MathConstant {
        uint Pi;             // π (Pi) ≈ 3.1415926535...
        uint Phi;            // Golden ratio ≈ 1.6180339887...
        uint Tau;            // Tau (2pi) ≈ 6.283185...
        uint Omega;          // Ω (Omega) ≈ 0.5671432904...
        uint ImaginaryUnit;  // i (Imaginary Unit) = √–1
        uint EulerNb;        // Euler number ≈ 2.7182818284590452...
        uint PythagoraConst; // Pythagora constant (√2) ≈ 1.41421... 
        uint TheodorusConst; // Theodorus constant (√3) ≈ 1.73205... 
    }   
}
```

```solidity
pragma solidity 0.8.3;
//calling the lib
import "https:github.com/OpenZepplin/openzepplin-contracts/contracts/math/SafeMath.sol";

contract LibrariesExample {

    using SafeMath for uint;   //using the lib for uint methods like .add/.sub(_amt)

    mapping(address=> uint) public tokenBalance;

    constructor() {
        tokenBalance[msg.sender] = 1;
    }

    function sendToken(address _to, uint _amt) public returns(bool){
        tokenBalance[msg.sender] -= tokenBalance.[msg.sender].sub(_amt);
        tokenBalance[_to] += tokenBalance.[msg.sender].add(_amt);

        return true;
    }
}
```

### Benefits <a href="#dbf7" id="dbf7"></a>

* **Usability:** Functions in a library can be used by many contracts. If you have many contracts that have some common code, then you can deploy that common code as a library.
* **Economical:** Using a base contract to deploy the common code won’t save gas because in Solidity, inheritance works by copying code. However, deploying common code as library will save gas&#x20;
  * as code is reused using the DELEGATECALL feature and execution is done within the context of the calling contract.&#x20;
* **Good add-ons :** Solidity libraries can be used to add member functions to data types. For instance, think of libraries like the _standard libraries_ or _packages_ that you can use in other programming languages to perform complex math operations on numbers.

### Limitations <a href="#84e2" id="84e2"></a>

Libraries in Solidity are considered **stateless,** and hence have the following restrictions:

* They **do not have any storage** (so can’t have non-constant state variables)
  * Can hold `struct` and `enum` : these are user-defined variables.
  * any other variable defined as `constant` (immutable), since constant variables are stored in the contract’s bytecode, not in storage.
* They **can’t hold ETH**&#x20;
  * can’t have a **fallback** function
  * Doesn’t allow payable functions&#x20;
* **Cannot inherit nor be inherited**
* **Can’t be destroyed** (no `selfdestruct()` function since version 0.4.20)

## How to deploy libraries?

Library deployment is a bit different from regular smart contract deployment. Here are two scenarios :

1. **Embedded Library:** If a smart contract is consuming a library which have only **internal functions,** then the EVM simply embeds library into the contract. Instead of using delegate call to call a function, it simply uses JUMP statement (normal method call). There is no need to separately deploy library in this scenario.
2. **Linked Library :** On the flip side, if a library contain **public or external functions** then library needs to be deployed. The deployment of library will generate a unique address in the blockchain. This address needs to be linked with calling contract.

#### Read More:

* [https://coinsbench.com/deep-dive-into-solidity-libraries-e9bd7f9061fb](https://coinsbench.com/deep-dive-into-solidity-libraries-e9bd7f9061fb)
