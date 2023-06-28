# enum

* Enums are one way to create a user-defined type in Solidity.
* Useful to model choice and keep track of state.
* They are integers internally.

```solidity
// 0-> Open state, 1 -> closed state, ...
enum LOTTERY_STATE {OPEN, CLOSED, CALCULATING_WINNER}
    
//init lottery_state as type: LOTTERY_STATE (is a class).
LOTTERY_STATE public lottery_state;   


constructor(address _pricefeedaddress) public {
    ethusd_pricefeed = AggregatorV3Interface(_pricefeedaddress);
    lottery_state = LOTTERY_STATE.CLOSED;       // can also: lottery_state = 1
}

// init outside connstructor, so its state variable. 
// then assign within constructor. 
// if you init + assign within constructor, it wont be state variable.

// example usage:
function enter() public payable {
        //is lottery open?
        require(lottery_state == LOTTERY_STATE.OPEN, "Lottery is not open!");
}
```

* create an enum class "LOTTERY\_STATE"
* init variable lottery\_state as type "LOTTERY\_STATE"
* assign lottery\_state LOTTERY\_STATE.CLOSED&#x20;

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Enum {
    // Enum representing shipping status
    enum Status {
        Pending,
        Shipped,
        Accepted,
        Rejected,
        Canceled
    }

    // Default value is the first element listed in
    // definition of the type, in this case "Pending"
    Status public status;

    // Returns uint
    // Pending  - 0
    // Shipped  - 1
    // Accepted - 2
    // Rejected - 3
    // Canceled - 4
    function get() public view returns (Status) {
        return status;
    }

    // Update status by passing uint into input
    function set(Status _status) public {
        status = _status;
    }

    // You can update to a specific enum like this
    function cancel() public {
        status = Status.Canceled;
    }

    // delete resets the enum to its first value, 0
    function reset() public {
        delete status;
    }
}
```

## Declaring and importing Enum <a href="#declaring-and-importing-enum" id="declaring-and-importing-enum"></a>

* Enums can be declared outside of a contract

File that the enum is declared in:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;
// This is saved 'EnumDeclaration.sol'

enum Status {
    Pending,
    Shipped,
    Accepted,
    Rejected,
    Canceled
}
```

File that imports the enum above:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

import "./EnumDeclaration.sol";

contract Enum {
    Status public status;
}
```
