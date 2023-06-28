# State Inheritance Testing

{% tabs %}
{% tab title="Initial Approach" %}
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

abstract contract StateZero is DSTest {
    SimpleNameRegister public simpleNameRegister;
    CheatCodes cheats;
    
    function setUp() public virtual {
        simpleNameRegister = new SimpleNameRegister();
        cheats = CheatCodes(HEVM_ADDRESS);
    }
}

contract StateZeroTest is StateZero {

    function testCannotRelease(string memory testString) public {  
        cheats.expectRevert(bytes("Not your name!"));
        simpleNameRegister.release(testString);
    }

    function testRegister(string memory testString) public {
        simpleNameRegister.register(testString);
        bool success = (address(this) == simpleNameRegister.holder(testString));
        assertTrue(success);
    }
}

abstract contract StateRegistered is StateZero {
    address adversary;
    string name;

    function setUp() public override {
        super.setUp();
        adversary = 0xE6A2e85916802210147e366D4431f5ca4dD51a78;
        
        // state transition
        name = 'whale';
        simpleNameRegister.register(name);
    }
}

contract StateRegisteredTest is StateRegistered {

    function testAdversaryCannotRegisterName() public {
        cheats.startPrank(adversary);
        cheats.expectRevert(bytes("Already registered!"));
        simpleNameRegister.register(name);   
        cheats.stopPrank();
    }

    function testAdversaryCannotReleaseName() public {
        cheats.startPrank(adversary);
        cheats.expectRevert(bytes("Not your name!"));
        simpleNameRegister.release(name);   
        cheats.stopPrank();
    }

    function testUserCannotRegisterOwnedName() public {
        cheats.expectRevert(bytes("Already registered!"));
        simpleNameRegister.register(name);
    }

    function testUserRelease() public {
        simpleNameRegister.release(name);
        bool success = (address(0) == simpleNameRegister.holder(name));
        assertTrue(success);
    }
}
```
{% endtab %}

{% tab title="SimpleNameRegister.sol" %}
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

///@title An on-chain name registry
///@author Calnix
contract SimpleNameRegister {
    
    /// @notice map a name to an address to identify current holder 
    mapping (string => address) public holder;    

    /// @notice emit an event when a name is registered or released
    event Register(address indexed holder, string name);
    event Release(address indexed holder, string name);

    /// @notice user can register an available name
    function register(string memory name) public {
        require(holder[name] == address(0), "Already registered!");
        holder[name] = msg.sender;
        emit Register(msg.sender, name);
    }

    /// @notice holder can release a name, making it available
    function release(string memory name) public {
        require(holder[name] == msg.sender, "Not your name!");
        delete holder[name];
        emit Release(msg.sender, name);
    }
}
```
{% endtab %}
{% endtabs %}

State setups and transitions will be realized by abstract contracts. Here we have two states.

1. StateZero -> Inception, nothing has been done.
2. StateRegistered -> User has registered a name.

In StateZero, we will have to instantiate SimpleNameRegister, to interact with the contract for testing. We opt to instantiate cheats here as well.

> abstract contract StateZero is DSTest{}

In StateRegistered, the user will register a name - which we observe in its setup function. Additionally, we instantiate an adversary address necessary for testing in this state.

### State Zero: Inception

#### Create the state (abstract contract):

* will contain the setUp() for State Zero
* instantiate SimpleNameRegister
* instantiate cheats (for use later in StateRegistered)

```solidity
abstract contract StateZero is DSTest {
    SimpleNameRegister public simpleNameRegister;
    CheatCodes cheats;
    
    function setUp() public virtual {
        simpleNameRegister = new SimpleNameRegister();
        cheats = CheatCodes(HEVM_ADDRESS);
    }
}
```

#### Create the test contract

* To execute the required tests in StateZero, create a contract StateZeroTest.&#x20;
* This will contain all test functions pertaining to said state.&#x20;
* The state will be realized by inheritance (StateZeroTest is StateZero).

```solidity
contract StateZeroTest is StateZero {

    function testCannotRelease(string memory testString) public {  
        cheats.expectRevert(bytes("Not your name!"));
        simpleNameRegister.release(testString);
    }

    function testRegister(string memory testString) public {
        simpleNameRegister.register(testString);
        bool success = (address(this) == simpleNameRegister.holder(testString));
        assertTrue(success);
    }
}
```

By the way of inheritance, the setUp function and state variables contained within StateZero will be executed setting up the environment for test functions belonging to StateZeroTest.

### State Transition

State transition occurs by inheritance between the abstract contracts.&#x20;

A movement from state(0) -> state(1) is reflected as `StateRegistered is StateZero`.&#x20;

```solidity
abstract contract StateRegistered is StateZero {
    address adversary;
    string name;

    function setUp() public override {
        super.setUp();
        adversary = 0xE6A2e85916802210147e366D4431f5ca4dD51a78;
        
        // state transition
        name = 'whale';
        simpleNameRegister.register(name);
    }
}
```

The initial state is inherited and executed (`super.setUp`), building upon it, are the necessary actions to evolve into StateRegistered.

In this this case, those actions would be a user registering a name:&#x20;

* name = 'whale';&#x20;
* simpleNameRegister.register(name);

Now that StateRegistered has been realized, similar to before, we will create a separate contract composing of the test functions pertaining to this state.

#### Create the test contract for StateRegistered

```solidity
contract StateRegisteredTest is StateRegistered {

    function testAdversaryCannotRegisterName() public {
        cheats.startPrank(adversary);
        cheats.expectRevert(bytes("Already registered!"));
        simpleNameRegister.register(name);   
        cheats.stopPrank();
    }

    function testAdversaryCannotReleaseName() public {
        cheats.startPrank(adversary);
        cheats.expectRevert(bytes("Not your name!"));
        simpleNameRegister.release(name);   
        cheats.stopPrank();
    }

    function testUserCannotRegisterOwnedName() public {
        cheats.expectRevert(bytes("Already registered!"));
        simpleNameRegister.register(name);
    }

    function testUserRelease() public {
        simpleNameRegister.release(name);
        bool success = (address(0) == simpleNameRegister.holder(name));
        assertTrue(success);
    }
}
```

This approach prizes modularity which could prove useful in complicated testing situations.

Additionally, tests from an earlier state will not be repeated in a subsequent state **as test contracts inherit state**, not test functions.
