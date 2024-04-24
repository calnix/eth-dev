# Router // batch

### Msg.sender

**Pool**

```solidity
contract Pool {

    uint256 public num;
    address public msgSender;

    function setVars(uint256 _num, address onBehalfOf) public {
        num = _num;
        msgSender = onBehalfOf;
    }
}
```

* msgSender here will be empty `0x0`, on deployment

**Router**

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.22;

contract Router {

    Pool public pool;

    /// @dev Helper function to extract a useful revert message from a failed call.
    /// If the returned data is malformed or not correctly abi encoded then this call can fail itself.
    function getRevertMsg(bytes memory returnData) internal pure returns (string memory)
    {
        // If the _res length is less than 68, then the transaction failed silently (without a revert message)
        if (returnData.length < 68) return "Transaction reverted silently";

        assembly {
            // Slice the sighash.
            returnData := add(returnData, 0x04)
        }

        return abi.decode(returnData, (string)); // All that remains is the revert string
    }

    /// @dev Allows batched call to self (this contract).
    /// @param calls An array of inputs for each call.
    function batch(bytes[] memory calls) public payable returns(bytes[] memory results) {

        results = new bytes[](calls.length);

        for (uint256 i; i < calls.length; i++) {

            (bool success, bytes memory result) = address(this).delegatecall(calls[i]);
            if (!success) revert(getRevertMsg(result));
            results[i] = result;
        }
    }


    function test(uint _num, address onBehalfOf) public {

        bytes memory payload = abi.encodeWithSignature("setVars(uint256,address)", _num, onBehalfOf);
        
        bytes[] memory allCalls = new bytes[](1);
        allCalls[0] = payload;

        batch(allCalls);
    }


    function setVars(uint _num, address onBehalfOf) external payable {
        pool.setVars(_num, onBehalfOf);
    }

    function setPool(address pool_) public {
        pool = Pool(pool_);
    }
}
```

* First we connect the Router to the Pool contract by calling **setPool** on the Router.
* Then we call `test`, passing in values for num and some address of choice.
* Notice that **`msgSender`** in Pool was updated with our input value.

Hence, the batch fn in Router, does pass on the correct input values through delegateCall.

### Use of msg.sender in Router

What if we pass msg.sender in batch, instead of defining it as an input address?

In this example, we modify **`test()`** to use msg.sender

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.22;

contract Router {

    Pool public pool;


    /// @dev Helper function to extract a useful revert message from a failed call.
    /// If the returned data is malformed or not correctly abi encoded then this call can fail itself.
    function getRevertMsg(bytes memory returnData) internal pure returns (string memory)
    {
        // If the _res length is less than 68, then the transaction failed silently (without a revert message)
        if (returnData.length < 68) return "Transaction reverted silently";

        assembly {
            // Slice the sighash.
            returnData := add(returnData, 0x04)
        }

        return abi.decode(returnData, (string)); // All that remains is the revert string
    }

    /// @dev Allows batched call to self (this contract).
    /// @param calls An array of inputs for each call.
    function batch(bytes[] memory calls) public payable returns(bytes[] memory results) {

        results = new bytes[](calls.length);

        for (uint256 i; i < calls.length; i++) {

            (bool success, bytes memory result) = address(this).delegatecall(calls[i]);

            if (!success) revert(getRevertMsg(result));

            results[i] = result;
        }

    }


    function test(uint _num) public {

        bytes memory payload = abi.encodeWithSignature("setVars(uint256,address)", _num, msg.sender);
        
        bytes[] memory allCalls = new bytes[](1);
        allCalls[0] = payload;

        batch(allCalls);
    }


    function setVars(uint _num, address onBehalfOf) external payable {
        pool.setVars(_num, onBehalfOf);
    }

    function setPool(address pool_) public {
        pool = Pool(pool_);
    }
}
```

```
contract Pool {

    uint256 public num;
    address public msgSender;

    function setVars(uint256 _num, address onBehalfOf) public {
        num = _num;
        msgSender = onBehalfOf;
    }
}
```

* msgSender in Pool reflects the wallet address that initiated the function.
* meaning to say, **`batch()`** in Router does forward the correct msg.sender value to Pool.&#x20;
