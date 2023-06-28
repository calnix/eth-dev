# caching sload into mload

## Cache read variables in memory

### Before:

```solidity
function liquidation(address user) external onlyOwner { 
    uint collateralRequired = getCollateralRequired(debts[user]);

    if (collateralRequired > deposits[user]){
        emit Liquidation(address(collateral), address(debt), user, debts[user], deposits[user]); 
        delete deposits[user];
        delete debts[user];
    }
```

Both `debts[user]` & `deposits[user]` are storage variables, given we are looking to pass a value from a mapping that was declared and as a public variable.

* A single SLOAD costs 2100 gas, and is unavoidable the first time.&#x20;
* The second instance, (line 5), we are reloading the SLOAD from cache and it will cost 100 gas.
* Hence the repetitive use of debts\[user] twice will cost us **2200** gas.

{% hint style="info" %}
Ever since [EIP-2929](https://eips.ethereum.org/EIPS/eip-2929) the first `SLOAD` operation costs 2100 gas, but once that memory is read, it is cached and considered considered warm, which has a cost of 100 gas to load again.
{% endhint %}

### After:

To combat that, you could always store the object in memory, and load it from there, which is much cheaper.&#x20;

So what you could do is write from storage to memory once (`SLOAD` + `MSTORE` = 2103 gas), then read the memory variable twice (`MLOAD` + `MLOAD`) = 6 gas.

* `SLOAD` + `MSTORE` = **2103 gas**
* `MLOAD` + `MLOAD` = 6 gas
* Total = **2109** gas

```solidity
function liquidation(address user) external onlyOwner { 
    uint userDebt = debts[user];             //saves an extra SLOAD
    uint userDeposit = deposits[user];       //saves an extra SLOAD

    require(!_isCollateralized(userDebt, userDeposit), "Not undercollateralized");

    delete deposits[user];
    delete debts[user];
    emit Liquidation(address(collateral), address(debt), user, userDebt, userDeposit); 
```
