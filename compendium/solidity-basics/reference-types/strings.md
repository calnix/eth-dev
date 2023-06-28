# strings

* strings are internally stored as bytes array
* they are then converted as UTF-8 again
* difficult to work with, and expensive.
* don't expect to work much with strings: instead use events!
* _need to specify with memory keyword when passing string as parameter_

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity 0.8.1;

contract StringExample {
    string public myString = 'hello world!';

    function setMyString(string memory _myString) public {
        myString = _myString;
    }

```

{% hint style="info" %}
of all the value types, string is the exception in that you are expected to declare memory when passing it as a parameter into a function.

The rest (u)int, address, boolean do not require any specification.
{% endhint %}

{% hint style="warning" %}
strings should be 32 bytes or less, otherwise they use 2 words in memory.
{% endhint %}

```solidity
require(nameOwner[_name] == address(0), "The provided name has already been registered!");
require(nameOwner[_name] == address(0), "Already registered!");
```

* The provided name has already been registered! -> 46 characters, totalling 46 bytes.
* `Already registered!` -> 19 characters, totalling 19 bytes

### Compact Strings

EVM stores data in 32 byte 'buckets' or 'slots'.

Strings are arrays of UTF-8 characters, and arrays are basically a fixed length sequence of storage slots located next to each other. Therefore, like everything else we need to think in terms of storage buckets

**This means that when we compile our contracts, we would prefer to use a single slot for the strings we use.**

Each character in a string is a UTF-8 Encoded byte, meaning that your strings can be up to 32 characters in length if you do not use specially encoded characters like emojis to be contained in a single storage slot.

{% hint style="info" %}
Here's a great resource to look up how many bytes a string takes up under UTF-8 Encoding. [https://mothereff.in/byte-counter](https://mothereff.in/byte-counter)\
[https://blog.hubspot.com/website/what-is-utf-8](https://blog.hubspot.com/website/what-is-utf-8)
{% endhint %}

### Equality comparison

[https://ethereum.stackexchange.com/questions/97394/check-if-calldata-contains-string](https://ethereum.stackexchange.com/questions/97394/check-if-calldata-contains-string)\
\
You cannot compare `bytes` or `string` using `==` in Solidity. You can however compare `bytes32`, so a simple solution is to hash the bytes first using `keccak256` and compare the result of that.

For example:

```php
function checker(bytes calldata _data) external pure returns (bool) {
  return keccak256(_data) == keccak256(abi.encodePacked("SOMETHING"));
}
```
