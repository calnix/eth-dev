# Renounce Ownership

Let's remove the function to remove an owner. This was inherited from Ownable.sol, but makes no sense here.

We simply stop this with a revert. Add the following function to the SharedWallet:

```solidity
contract SharedWallet is Allowance {

    //...

    function renounceOwnership() public override onlyOwner {
        revert("can't renounceOwnership here"); //not possible with this smart contract
    }

    //...
}
```

![](<../../../.gitbook/assets/image (277).png>)
