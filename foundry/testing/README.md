---
description: >-
  Forge is a command-line tool that ships with Foundry. Forge tests, builds, and
  deploys your smart contracts.
---

# Testing

### **Test Contracts**

* Testing in Solidity is different from other frameworks like brownie.
* Tests are written in in Solidity -> If the test function reverts, the test fails, otherwise it passes.
  * Essentially we are writing a separate contract containing test functions.
  * Test contracts end with `.t.sol`
  * Usually, tests will be placed in `src/test` (or `./test` in windows)&#x20;

#### Test contracts are deployed to `0xb4c79daB8f259C7Aee6E5b2Aa729821864227e84`.&#x20;

* If you deploy a contract within your test, then `0xb4c...7e84` will be its deployer.&#x20;
* If the contract deployed within a test gives special permissions to its deployer, such as `Ownable.sol`'s `onlyOwner` modifier, then the test contract `0xb4c...7e84` will have those permissions.

### Assertions

{% hint style="info" %}
Assertion references:&#x20;

* [https://book.getfoundry.sh/reference/ds-test.html](https://book.getfoundry.sh/reference/ds-test.html)
* [https://book.getfoundry.sh/reference/forge-std/index.html](https://book.getfoundry.sh/reference/forge-std/index.html)
{% endhint %}

### **Running Tests**

* Forge can run your tests with the [`forge test`](https://book.getfoundry.sh/reference/forge/forge-test.html) command.&#x20;
* Forge will look for the tests anywhere in your source directory.&#x20;
* Any contract with a function that starts with `test` is considered to be a test.&#x20;

**Running a specific test function**

* forge test --match-contract ComplicatedContractTest --match-test testDeposit
* run the tests in the test contract `ComplicatedContractTest` with `testDeposit` in the function name.

{% hint style="info" %}
Inverse versions of these flags also exist:

* `--no-match-contract`&#x20;
* `--no-match-test`
{% endhint %}

### Implementation

```solidity
pragma solidity ^0.8.13;

import "ds-test/test.sol";
import 'src/SimpleNameRegister.sol';

contract SimpleNameRegisterTest is DSTest {
    //declare state var.
    SimpleNameRegister simpleNameRegister;
    address owner;
    string test;

    function setUp() public {
        simpleNameRegister = new SimpleNameRegister();
        owner = 0xb4c79daB8f259C7Aee6E5b2Aa729821864227e84; //test contract deployer   
        test = "what";        
    }
    function testRegisterDepends() public {
        simpleNameRegister.registerName(test);
        bool success = (owner == simpleNameRegister.nameOwner(test));
        assertTrue(success);
    }

    function testRelinquishDepends() public {
        simpleNameRegister.relinquishName(test);
        bool success = (simpleNameRegister.nameOwner(test) == address(0));
        assertTrue(success);
    }
```

* import the contract to be tested  (import 'src/SimpleNameRegister.sol';)
* deploy it via factory pattern&#x20;

![factory pattern](<../../.gitbook/assets/image (180).png>)

#### function setUp()

The setup function is invoked before each test case is run:&#x20;

* serves to setup the necessary variables and conditions for your test functions to operate.
* here we use it to deploy a fresh instance of SimpleNameRegister.sol before each test case is ran via `simpleNameRegister = new SimpleNameRegister()`

{% hint style="info" %}
Each test is run as independent cases -> changes made in a prior test function will not spill out of scope into a following test function
{% endhint %}

#### **To show that there is no spillover:**

* create storage string variable test and assign it "what"
* The first function testRegisterDepends() registers ownership of name "what".
* Then we call testRelinquishDepends() to check if we can relinquish ownership.

![](<../../.gitbook/assets/image (93).png>)

testRelinquishDepends() fails, indicating that the effects from testRegisterDepends() do not spill into testRelinquishDepends().

Since setUp() is invoked before each test case -> each test function interacts with a seperate instance of SimpleNameRegister.sol

### Cheatcodes

Reference: [https://book.getfoundry.sh/cheatcodes/](https://book.getfoundry.sh/cheatcodes/)

* manipulate the state of the blockchain -> change block number
* change your identity
* test for specific reverts and events.

Cheatcodes are functions on contract at address: `0x7109709ECfa91a80626fF3989D68f67F5b1DD12D`

To execute a cheatcode function, we need to interface with this contract from our test contract.

**Therefore in our test contract we need to do 3 things:**

1. &#x20;import interface Cheatcodes{}  _(or just copypasta the interface above your contract like below)_
2. declare interface object as state variable
3. assignment of state variable: pass the cheatcode contract address into the interface object

![](<../../.gitbook/assets/image (240).png>)

{% hint style="info" %}
If you are using `ds-test`, then this address is assigned in a constant named `HEVM_ADDRESS`. No need to remember the full address.
{% endhint %}

#### Implementing Cheatcode: Prank

* `prank`: changes the next call's msg.sender to the input address.
* This allows us to modify msg.sender in between function calls.

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "ds-test/test.sol";
import 'src/SimpleNameRegister.sol';

interface CheatCodes {
    function prank(address) external;
    function expectRevert(bytes calldata) external;
    function startPrank(address) external;
    function stopPrank() external;
}

contract SimpleNameRegisterTest is DSTest {
    
    // declare state var.
    SimpleNameRegister simpleNameRegister;
    CheatCodes cheats;
    address adversary;

    function setUp() public {
        simpleNameRegister = new SimpleNameRegister();
        cheats = CheatCodes(HEVM_ADDRESS);
        adversary = 0xE6A2e85916802210147e366D4431f5ca4dD51a78;
    }

    // user can register an available name
    function testRegisterName(string memory _testString) public {
        simpleNameRegister.registerName(_testString);
        bool success = (address(this) == simpleNameRegister.nameOwner(_testString));
        assertTrue(success);
    }
    
    // user can register an available name and relinquish it
    function testRelinquishName(string memory _testString) public {
        simpleNameRegister.registerName(_testString);   
        simpleNameRegister.relinquishName(_testString);
        bool success = (simpleNameRegister.nameOwner(_testString) == address(0));
        assertTrue(success);
    }

    // user cannot relinquish a name that does not belong to them
    function testRelinquishAsNotOwner(string memory _testString) public {
        simpleNameRegister.registerName(_testString);   
        cheats.startPrank(adversary);
        cheats.expectRevert(bytes("The provided name does not belong to you!"));
        simpleNameRegister.relinquishName(_testString);        
        cheats.stopPrank();
    }
    
    // user cannot register a name that already has an owner
    function testRegisterUnavailableName(string memory _testString) public {
        simpleNameRegister.registerName(_testString);   
        cheats.startPrank(adversary);
        cheats.expectRevert(bytes("The provided name has already been registered!"));
        simpleNameRegister.registerName(_testString);   
        cheats.stopPrank();
    }
}
```

#### **startPrank**

![startPrank](<../../.gitbook/assets/image (267).png>)

Sets `msg.sender` **for all subsequent calls** until [`stopPrank`](https://book.getfoundry.sh/cheatcodes/stop-prank.html) is called.

**stopPrank**

![stopPrank](<../../.gitbook/assets/image (235).png>)

Stops [`startPrank`](https://book.getfoundry.sh/cheatcodes/start-prank.html), resetting `msg.sender` and `tx.origin` to the values before `startPrank` was called**.**

**expectRevert**

We can do so using `expectRevert(bytes calldata)`; it will expect a revert to occur on the **next call.**

* insert it before making a function call that is expected to fail/revert
* if the **next call** does not revert**,** then `expectRevert` will.
* After calling `expectRevert`, calls to other cheatcodes before the reverting call are ignored.

For require statements, provide the error string as defined in the require:

```solidity
cheats.expectRevert(bytes("The provided name does not belong to you!"));
```

{% hint style="info" %}
If we had used prank instead of start/stop prank, the test function would not have worked as prank **only** modifies the **next** function call.&#x20;

Therefore it would not have worked in this scenario as the next call is `cheats.expectRevert()` before `simpleNameRegister.registerName(_testString)`
{% endhint %}
