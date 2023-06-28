# Function signature + selectors

## Function Signature

A function signature is a combination of a function and the types of parameters it takes, combined together as a string with no spaces.

For example, let's say you have a function in solidity where the method looks like this:

```javascript
function transfer(address sender, uint256 amount) public {
  // Some code here
}
```

This function would have a function signature of:

```
transfer(address,uint256)
```

These are important because we use function signatures to get the next part, function selectors. Additionally, you'll want to use `uint256` instead of `uint` [for computing a function signature](https://docs.soliditylang.org/en/develop/abi-spec.html#types) (and function selector).

## Function Selector

A function selector is the first 4 bytes of the call data for a function call that specifies the function to be called. A function selector is the hash of the same function's signature.

Which might be a little confusing, but let's break it down. When someone makes a call to an EVM smart contract, the smart contract needs to know which function it should execute. The piece of code that governs this is known as the function selector, and it might look like this:

```javascript
0xa9059cbb //this is the function selector for the transfer function signature above. 
```

You can get the function selector by hashing the string of the function signature in solidity.

```javascript
bytes4(keccak256(bytes(function_signature)))
```

### Bonus

There are [EVM Signature Databases](https://sig.eth.samczsun.com/) that make it easier to find out the function signature of a selector. Understanding function signatures and selectors allow developers to [call functions of any contract without having an ABI](https://twitter.com/PatrickAlphaC/status/1517156225670078465).
