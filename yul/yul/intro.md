# Intro

#### From solidity to Yul

```solidity
function getNumer() external pure returns(uint256) {
    return 42;
}

// in Yul

function getNumber() external pure returns(uint256) {
    uint256 x;
    
    assembly{
        x := 42
    }

    return x;
}
```

* yul doesn not have semi-colons
* assignment within assembly block uses " **:= "**
* assembly code has visibility on local variables within function scope -> can "see" x, which was defined outside.

### Assembly has only 1 type: the 32byte word

```solidity
function getHex() external pure returns (uint256) {
    uint256 x;
    
    assembly{
        x := 0xa
    }
    
    return x;
}
```

* on compiling, x will return **10**
* Hexadecimal `0xa` is  decimal 10
* Solidity is interpretating it as a decimal, when casted as uint256

#### Negative Example: String

```solidity
function demoString() external pure returns (string memory) {
    string memory myString = "";
    
    assembly{
        myString := "hello world"
    }
    
    return myString;
}
```

* contract will compile, but calling demoString will result in an error
* myString was initialised as a memory variable -> stored on the heap (not the stack)
* the assignment within the assembly block is trying to assign a value to the pointer on the stack to a location in memory

#### Psitive Example: String

```solidity
function demoString() external pure returns (bytes32) {
    bytes32 myString = "";
    
    assembly{
        myString := "hello world"
    }
    
    return myString;
    return string(abi.encode(myString));
}
```

* if we use `return myString`solidity will return a hexadecimal interpretations
* on abi.ecode we will get back the human readable version

{% hint style="info" %}
**Gotcha!**\
Assembly assumes that the string value is <= 32 bytes. \
We will get to using longer strings later.
{% endhint %}
