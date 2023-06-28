# Shadowing in Fuctions

Variables in solidity can be shadowed, meaning they can be declared multiple times within the same scope with the same name, but with different values. This can lead to unexpected results, particularly when the variable is used in a function.

```solidity
pragma solidity 0.8.15;

contract Shadowing {
    uint n = 2;
    uint x = 3;

    function test1() public view returns (uint n) {
        return n; // Will return 0
    }

    function test2() public view returns (uint n) {
        n = 1;
        return n; // Will return 1
    }

    function test3() public view returns (uint x) {
        uint n = 3;
        return n+x; // Will return 3
    }

    function test4() public view returns (uint) {
        return n; // Will return 2
    }  
}
```

**Why does the value of `n` test 1 and 4 differ?**

* test 1: `n` behaves as if it has been assigned the value of `0`, &#x20;
* test 4: `n` behaves like the storage variable declared at the start of the contract.

**In test 1, n is declared as a local variable, within the returns type.**&#x20;

* Local variables take precedence over storage variables in the case of shadowing.&#x20;
* Local variable `n`  is declared without assignment.&#x20;
* An unassigned uint variable will be of the default value `0`.

**We see this again in test 3 with `x`**

* `x == 0, n == 3`
* Both variables are locally declared.
* However, x does not have an assignment, and defaults to 0.

### Gotcha!&#x20;

```solidity
// ... extending earlier contract
function test5() public view returns (uint x) {
    uint x = 7; // will return 0
}

function test6() public view returns (uint x) {
    x = 7;  // will return 7
}
```

test6 works as expected since there is no shadowing involved. test5 returns `0` because:

* Within the function body, a new unsigned integer variable `x` is declared and assigned the value of 7.
* However, this variable declaration is shadowing the return value declaration of `x` from the function signature.
* **Return value declaration takes precedence!**&#x20;

{% hint style="info" %}
This gotcha was contributed - courtesy of [devtooligan](https://twitter.com/devtooligan)
{% endhint %}
