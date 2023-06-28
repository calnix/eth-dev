# Custom errors over require

The benefit of custom errors is that they can significantly reduce the cost to deploy and call a contract, while still providing the same amount of information (or more). Let's look at the following example:

```solidity
pragma solidity ^0.8.4;

contract Example {
  error InvalidSenderAddress(address _address);

  function foo() external {
    revert InvalidSenderAddress(msg.sender);
  }
}
```

* This costs 83,249 gas to deploy in Remix, and calling `foo` results in a gas cost of 21,245. Now let's compare that to the following contract:

```solidity
pragma solidity ^0.8.4;

contract Example {
  function foo() external {
    revert("Invalid sender address");
  }
}
```

* This costs 91,463 gas to deploy, and calling `foo` results in a gas cost of 21,288, simply because we are using a string instead of a custom error. It also does not provide any info, like which address is invalid for example.
* Custom errors are ABI encoded, and can be decoded using existing ABI decoders. This makes it a lot more efficient to store and use compared to strings.
* For a more in-depth explanation of custom errors: [https://blog.soliditylang.org/2021/04/21/custom-errors/](https://blog.soliditylang.org/2021/04/21/custom-errors/)
