# Project #1: Simple Registry

## Objectives

* Users (identified by an address) can claim a name, which is recorded on-chain.
* Once a name has been claimed, no other user can claim it.
* A name owner can release a name.
* A user can claim any number of names.

## Blueprint

* A mapping to serve as a registry.
* Function to register names.
* Function to release ownership of a name.
* Events to announce state changes for each function.

### Base Code

{% code title="SimpleNameRegister.sol" %}
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

/**
@title An on-chain name registry
@author Calnix
@notice A registed name can be released, availing to be registered by another user
*/
contract SimpleNameRegister {
    
    /// @notice Map a name to an address to identify current holder 
    mapping (string => address) public holder;    

    /// @notice Emit event when a name is registered
    event Register(address indexed holder, string name);

    /// @notice Emit event when a name is released
    event Release(address indexed holder, string name);

    /// @notice User can register an available name
    /// @param name The string to register
    function register(string calldata name) external {
        require(holder[name] == address(0), "Already registered!");
        holder[name] = msg.sender;
        emit Register(msg.sender, name);
    }

    /// @notice Holder can release a name, making it available
    /// @param name The string to release
    function release(string calldata name) external {
        require(holder[name] == msg.sender, "Not your name!");
        delete holder[name];
        emit Release(msg.sender, name);
    }
}
```
{% endcode %}

#### 1. Mapping: associating names to their respective owner's address

```solidity
mapping (string => address) public holder;    
```

* initialized as a public state variable.
* solidity will automatically create a getter function for it, which we can use to pass a name as parameter to check ownership.
* If there is no owner, the address returned will be `0x0000000000000000000000000000000000000000`&#x20;

#### 2. Function to allow a user to register their ownership of a name

```solidity
    function register(string calldata name) external {
        require(holder[name] == address(0), "Already registered!");
        holder[name] = msg.sender;
        emit Register(msg.sender, name);
    }
```

* require statement checks if the name passed is available to be claimed.
* If available, passed name is mapped to the address of the `msg.sender` via the mapping `holder`
* event `Registered` emitted each time a name is registered.

#### 3. Function to allow a user to relinquish their ownership of a name

```solidity
    function release(string calldata name) external {
        require(holder[name] == msg.sender, "Not your name!");
        delete holder[name];
        emit Release(msg.sender, name);
    }
```

* require statement checks if the name passed is indeed registered to the function caller (msg.sender)
* If so, we reset the mapped address of the name to the zero address.
  * _mappings can be seen as hash tables which are virtually initialized such that every possible key exists and is mapped to default values._
  * _the default value for address type will be_ `0x0000000000000000000000000000000000000000` or  `address(0)`
* event `Release`emitted each time a name is relinquished.

## Events

Events allow us to “print” information on the blockchain in a way that is more searchable and gas efficient than just saving to public storage variables in our smart contracts.

{% hint style="info" %}
For more on events: [https://calnix.gitbook.io/solidity-lr/return-and-events#events](https://calnix.gitbook.io/solidity-lr/return-and-events#events)
{% endhint %}

### **Problem:**

* A single address could be the owner of multiple names.
* It would be useful if the end-user could simply supply their address to check which are the names registered to them.
* Achieving this one to many structure on-chain would not be gas efficient.

### **Solution:  Using Events**

* Run a background task that subscribes to events from the contract.&#x20;
* This background task will listen to events as they are emitted and track the list of addresses for each owner via some centralized database.&#x20;
* Website front-end can read from this database and reflect accordingly.&#x20;

## Testing

Testing in Solidity is somewhat different from other frameworks like brownie, as they would be done in Solidity. Essentially we deploy a test contract containing our test functions.&#x20;

{% tabs %}
{% tab title="Linear Test Approach" %}
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
{% endtab %}

{% tab title="State Inheritance Approach" %}
```solidity
pragma solidity ^0.8.13;

import "ds-test/test.sol";
import 'src/SimpleNameRegister.sol';

import "forge-std/console2.sol";

interface CheatCodes {
    function prank(address) external;
    function expectRevert(bytes calldata) external;
    function label(address addr, string calldata label) external;
}

abstract contract StateZero is DSTest {
    SimpleNameRegister public simpleNameRegister;
    CheatCodes cheats;
    address user;
    
    function setUp() public virtual {
        simpleNameRegister = new SimpleNameRegister();
        cheats = CheatCodes(HEVM_ADDRESS);
        user = 0x0000000000000000000000000000000000000001;
        cheats.label(user, "user");
    }
}

contract StateZeroTest is StateZero {

    function testCannotRelease(string memory testString) public {  
        console2.log("Cannot release a name that you do not hold");
        cheats.prank(user);
        cheats.expectRevert(bytes("Not your name!"));
        simpleNameRegister.release(testString);
    }

    function testRegister(string memory testString) public {
        console2.log("User registers a name");
        cheats.prank(user);
        simpleNameRegister.register(testString);
        bool success = (user == simpleNameRegister.holder(testString));
        assertTrue(success);
    }
}

abstract contract StateRegistered is StateZero {
    address adversary;
    string name;

    function setUp() public override {
        super.setUp();
        adversary = 0xE6A2e85916802210147e366D4431f5ca4dD51a78;
        cheats.label(adversary, "adversary");
        
        // state transition
        name = 'whale';
        cheats.prank(user);
        simpleNameRegister.register(name);
    }
}

contract StateRegisteredTest is StateRegistered {

    function testAdversaryCannotRegisterName() public {
        console2.log("Adversary cannot register name belonging to User");
        cheats.prank(adversary);
        cheats.expectRevert(bytes("Already registered!"));
        simpleNameRegister.register(name);   
    }

    function testAdversaryCannotReleaseName() public {
        console2.log("Adversary cannot release name belonging to User");
        cheats.prank(adversary);
        cheats.expectRevert(bytes("Not your name!"));
        simpleNameRegister.release(name);   
    }

    function testUserCannotRegisterOwnedName() public {
        console2.log("User cannot register a name that he already holds");
        cheats.prank(user);
        cheats.expectRevert(bytes("Already registered!"));
        simpleNameRegister.register(name);
    }

    function testUserRelease() public {
        console2.log("User releases name that he holds");
        cheats.prank(user);
        simpleNameRegister.release(name);
        bool success = (address(0) == simpleNameRegister.holder(name));
        assertTrue(success);
    }
}
```
{% endtab %}
{% endtabs %}

#### function setUp()

The setup function is invoked before each test case is run. Serves to setup the necessary variables and conditions for your test functions to operate.

* deploy a fresh instance of SimpleNameRegister before each test function is ran via `simpleNameRegister = new SimpleNameRegister()`
* `address adversary` will be used in test 3 & 4

{% hint style="info" %}
Each test is run as independent cases -> changes made in a prior test function will not spill out of scope into a following test function (See more: [https://calnix.gitbook.io/solidity-lr/foundry/testing#to-show-that-there-is-no-spillover](https://calnix.gitbook.io/solidity-lr/foundry/testing#to-show-that-there-is-no-spillover))
{% endhint %}

{% hint style="info" %}
For more about CheatCodes: [https://app.gitbook.com/s/Tgomzlmn9NrxUY0OQ3cD/foundry/testing#cheatcodes](../foundry/testing/#cheatcodes)
{% endhint %}

### Linear vs State Inheritance Approach

[https://calnix.gitbook.io/solidity-lr/yield-mentorship-2022/simple-registry-1/state-inheritance-testing](https://calnix.gitbook.io/solidity-lr/yield-mentorship-2022/simple-registry-1/state-inheritance-testing)

## Deployment

```bash
forge create src/SimpleNameRegister.sol:SimpleNameRegister --private-key ${PRIVATE_KEY_EDGE} --rpc-url ${ETH_RPC_URL}
```

* Forge can deploy only one contract at a time.
* Solidity files may contain multiple contracts. `:MyContract` above specifies which contract to deploy from the `src/MyContract.sol` file

### Verifying

```bash
forge verify-contract --chain-id ${KOVAN_CHAINID} --compiler-version v0.8.13+commit.abaa5c0e ${CONTRACT_ADDRESS} src/SimpleNameRegister.sol:SimpleNameRegister ${ETHERSCAN_API_KEY} --num-of-optimizations 200
```

{% hint style="info" %}
`Set --num-of-optimizations 200`

If not set on deployment, foundry defaults to 200. So make sure you pass `--num-of-optimizations 200` if you left the default compilation settings
{% endhint %}

### Check verification

```bash
forge verify-check --chain-id ${KOVAN_CHAINID} ${GUID} ${ETHERSCAN_API_KEY}
```

Running the above three commands manually, each time can be bothersome. To that end, create a new file 'MakeFile' in the project root directory if one does not exist.

{% tabs %}
{% tab title="Makefile" %}
```bash
# include .env file and export its env vars (-include to ignore error if it does not exist)
include .env

deploy:
	forge create src/SimpleNameRegister.sol:SimpleNameRegister --private-key ${PRIVATE_KEY_EDGE} --rpc-url ${ETH_RPC_URL}

verify:
	forge verify-contract --chain-id ${KOVAN_CHAINID} --compiler-version v0.8.13+commit.abaa5c0e ${CONTRACT_ADDRESS} src/SimpleNameRegister.sol:SimpleNameRegister ${ETHERSCAN_API_KEY} --num-of-optimizations 200 --flatten

verify-check:
	forge verify-check --chain-id ${KOVAN_CHAINID} ${GUID} ${ETHERSCAN_API_KEY}

```

Now you will only need to run:

* make deploy
* make verify
* make verify-check

{% hint style="danger" %}
Be sure to update the contract address and GUID values in .env for each deployment as they would change.&#x20;

Note:\
For make verify, the --flatten flag may ONLY be necessary on a window machine. If verification fails, try without it.\
\
If it still fails, its in god's hands now.
{% endhint %}
{% endtab %}

{% tab title=".env" %}
```bash
export PRIVATE_KEY_EDGE = 0x1232141356361...42414
export ETH_RPC_URL = https://kovan.infura.io/v3/b2a555d8b6e64178bbb736aabaf93a1a
export ETHERSCAN_API_KEY = I5CK8CJ7FZ6BWQ6YGFD4U3E7CFTNWQJ3A3
export Contract = src/SimpleNameRegister.sol:SimpleNameRegister
export KOVAN_CHAINID = 42

export CONTRACT_ADDRESS = 0xa155f6b896d1a22b2c18cb6229e4e0760176199b
export GUID = dyrsthjrhefssx6epi91ky8dxdun69ufbmz5rgqt6dzxzcgt4w
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Installing make on windows:\
[https://stackoverflow.com/questions/32127524/how-to-install-and-use-make-in-windows](https://stackoverflow.com/questions/32127524/how-to-install-and-use-make-in-windows)
{% endhint %}
