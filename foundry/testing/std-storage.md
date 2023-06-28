---
description: https://book.getfoundry.sh/reference/forge-std/std-storage.html
---

# std-storage

## stdstore

**Query functions:**

* `target`: Set the address of the contract (required)
* `sig`: Set the signature of the function to call (required)
* `with_key`: Set the mapping key, if the variable is of type `mapping`
* `depth`: Set the index of the struct member, if the variable is of type `struct`

**Terminator functions:**

* `checked_write`: Set the data to be written to the storage slot(s)
* `find`: Return the slot number

### Example 1

`playerToCharacter` tracks each player's character's stats.

```solidity
// MetaRPG.sol

struct Character {
    Class class;
    uint256 level;
    uint256 xp;
}

mapping (address => Character) public playerToCharacter;
```

Let's say we want to set the level of our character to 120.

```solidity
// MetaRPG.t.sol

stdstore
    .target(address(metaRpg))
    .sig(metaRpg.playerToCharacter.selector)
    .with_key(address(this))
    .depth(1)
    .checked_write(120);
```

### Example 2

```solidity
function setUp() public virtual {
        dai = new Dai(4);
        vm.label(address(dai), "dai contract");

        weth = new WETH9();
        vm.label(address(weth), "weth contract");

        //mint 100 DAI for Vault
        vault = new Vault(address(dai), address(weth), address(priceFeed),8,10);
        vm.label(address(vault), "vault contract");
        
        // dai.mint(address(vault), 10000*10**18);
        //......accessing stdstore instance........\\
        stdstore    
        .target(address(dai))
        .sig(dai.balanceOf.selector)    //select balanceOf mapping
        .with_key(address(vault))       //set mapping key balanceOf(address(vault))
        .checked_write(10000*10**18);   //data to be written to the storage slot -> balanceOf(address(vault)) = 10000*10**18
```

## .selector

* [https://ethereum.stackexchange.com/questions/72687/explanation-of-appending-selector-in-solidity-smart-contracts](https://ethereum.stackexchange.com/questions/72687/explanation-of-appending-selector-in-solidity-smart-contracts)
* A function selector is the first four bytes of the calldata that specifies which function to call during a function call

`.selector` returns the [ABI function selector.](https://solidity.readthedocs.io/en/latest/abi-spec.html#abi-function-selector)

> The first four bytes of the call data for a function call specifies the function to be called. **It is the first (left, high-order in big-endian) four bytes of the Keccak-256 (SHA-3) hash of the signature of the function.** **The signature is defined as** the canonical expression of the basic prototype without data location specifier, i.e. **the function name with the parenthesised list of parameter types**. Parameter types are split by a single comma - **no spaces are used**.

* [https://ethereum.stackexchange.com/questions/72687/explanation-of-appending-selector-in-solidity-smart-contracts](https://ethereum.stackexchange.com/questions/72687/explanation-of-appending-selector-in-solidity-smart-contracts)
* [https://ethereum.stackexchange.com/questions/72363/what-is-a-function-selector](https://ethereum.stackexchange.com/questions/72363/what-is-a-function-selector)

