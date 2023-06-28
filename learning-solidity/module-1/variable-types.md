# Variable Types

## Data types

Solidity supports a variety of data types, including:

* Boolean: `bool`
* Integer: `int` and `uint` of various sizes
* Address: `address`
* Bytes: `bytes` and `byte`
* String: `string`
* Arrays: `array`
* Structs: `struct`
* Enumerations: `enum`

## Default values&#x20;

The concept of “undefined” or “null” values does not exist in Solidity. Declared and unassigned variables have a [default value](https://docs.soliditylang.org/en/latest/control-structures.html#default-value) dependent on its type.

* (u)int = `0`
* bool = `False`
* string = `""`

## Examples

### Boolean

The bool type in Solidity can have one of two values: true or false. It is used to represent conditions that can be either true or false. Here's an example:

```solidity
bool isReady = true;
```

### Integer

The int and uint types in Solidity are used to represent signed and unsigned integers, respectively. The size of the integer can vary depending on the number of bits used to represent it. Here are some examples:

```solidity
int8 myInt8 = -10; 
uint256 myUint256 = 1000;
```

### Address

The address type in Solidity is used to represent Ethereum addresses. An Ethereum address is a 20-byte value that represents an account on the Ethereum blockchain. Here's an example:

```solidity
address myAddress = 0x1234567890123456789012345678901234567890;
```

### Bytes

The bytes type in Solidity is used to represent a dynamic array of bytes. The byte type is used to represent a single byte. Here are some examples:

```solidity
bytes memory myBytes = new bytes(10); 
byte myByte = 0x12;
```

### String

The string type in Solidity is used to represent a dynamic array of characters. Here's an example:

```solidity
string memory myString = "Hello, World!";
```

### Arrays

Solidity supports both fixed-size and dynamic arrays. Here's an example of a fixed-size array:

```solidity
uint256[3] myArray = [1, 2, 3];
```

And here's an example of a dynamic array:

```solidity
uint256[] myDynamicArray;
```

## Structs

A struct is a custom data type that allows you to define a collection of variables with different data types. Here's an example:

```solidity
struct Person { 
    string name; 
    uint256 age; 
} 

Person myPerson = Person("Alice", 25);
```

## Enumerations

An enumeration is a custom data type that allows you to define a set of named values. Here's an example:

<pre class="language-solidity"><code class="lang-solidity"><strong>enum Status {
</strong>    Pending,
    Shipped,
    Accepted,
    Rejected,
    Canceled
} 

    // Returns uint
    // Pending  - 0
    // Shipped  - 1
    // Accepted - 2
    // Rejected - 3
    // Canceled - 4

// Default value is the first element listed: "Pending"
Status public status;

// returns 0
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
</code></pre>
