---
description: >-
  https://github.com/RareSkills/Udemy-Yul-Code/blob/main/Video-03-Basic-Operations.sol
---

# Basic operations

## Operations

```solidity
// The rest:
    /*
        | solidity | YUL       |
        +----------+-----------+
        | a && b   | and(a, b) |
        +----------+-----------+
        | a || b   | or(a, b)  |
        +----------+-----------+
        | a ^ b    | xor(a, b) |
        +----------+-----------+
        | a + b    | add(a, b) |
        +----------+-----------+
        | a - b    | sub(a, b) |
        +----------+-----------+
        | a * b    | mul(a, b) |
        +----------+-----------+
        | a / b    | div(a, b) |
        +----------+-----------+
        | a % b    | mod(a, b) |
        +----------+-----------+
        | a >> b   | shr(b, a) |
        +----------+-----------+
        | a << b   | shl(b, a) |
        +----------------------+
    */
```

### For loop

```solidity
function isPrime(uint256 x) public pure returns (bool p) {
    p = true;
    
    assembly {
        //uint256 halfX = (x/2) +1
        let halfX := add(div(x,2), 1)
        
//     startCondition | stopCondition | counter update
        for {let i := 2} lt(i, halfX) {i := add(i,1)}
        {
            if iszero(mod(x,i)){
                p:=0
                break       // if mod is 0, number is not prime. set return value to 0
                            // p:=0, returns false
            }
        }
        
    }


}
```

* function checks is a number is prime
* it does this by trying every number between 2 and half of the number you are testing, and see if the modulus is zero at any point

### If loop

since Yul only has the 32byte word as a type, there are no booleans. Evaluating true/false is going to be different.

```solidity
function isTruthy() external pure returns (uint256 result) {
    result = 2;

    assembly{
        if 2 {
            result := 1    
        }
    }
    returns result;  //returns 1
}

function isFalsy() external pure returns (uint256 result) {
    result = 1;
    
    assembly {
        if 0 {
            result := 2
        }
    }
    
    returns results //returns 1
}
```

* 1st scenario: if `2` is considered "`true`", result will be returned as `1`
* 2nd scenario: if 0 evaluates to `false`

{% hint style="info" %}
In Yul, falsy values are when all the 32 byte word are **0.**
{% endhint %}

### Negation

```solidity
function negation() external pure returns (uint256 result) {
    result = 1;
    assembly {
        if iszero(0) {
            result := 2
        }
    }

    return result; // returns 2
}
```

* for negation use `iszero`
* unsafe negation: avoid using the `not`() keyword

**What happens with not()**

```solidity
function bitFlip() external pure returns (bytes32 result) {
    assembly {
        result := not(2)
    }
}
```

* not will simply flip all the bits, and in the case of 2, you will get a non-zero value back.
* this results in `if not(<non-zero>)` evaluating to true. See example:

{% tabs %}
{% tab title="unsafe2NegationPart" %}
```solidity
function unsafe2NegationPart() external pure returns (uint256 result) {
    result = 1;
    assembly {
        if not(2) {
            result := 2
        }
    }

    return result; // returns 2
}
```
{% endtab %}

{% tab title="unsafe1NegationPart1" %}
```solidity
function unsafe1NegationPart1() external pure returns (uint256 result) {
    result = 1;
    assembly {
        if not(0) {
            result := 2
        }
    }

    return result; // returns 2
}
```
{% endtab %}
{% endtabs %}

