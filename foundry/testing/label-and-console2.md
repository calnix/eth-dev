# label & console2

### vm.label

```solidity
function setUp() public virtual {
    //vm.chainId(4);
    dai = new Dai(4);
    vm.label(address(dai), "dai contract");

    weth = new WETH9();
    vm.label(address(weth), "weth contract");

    priceFeed = new MockV3Aggregator(18, 386372840000000);
    vm.label(address(priceFeed), "priceFeed contract");

    deployer = 0xb4c79daB8f259C7Aee6E5b2Aa729821864227e84;
    vm.label(deployer, "deployer");

    user = address(1);
    vm.label(user, "user");
```

### console2.log

```solidity
function testVaultHasDAI() public {
    console2.log("Check Vault has 100 DAI on deployment");
    uint vaultDAI = dai.balanceOf(address(vault));
    assertTrue(vaultDAI == 10000*10**18);
}
```

![label and console2.log](<../../.gitbook/assets/image (57).png>)

{% hint style="info" %}
logging doesnt get reflected in fuzzing tests.
{% endhint %}

* vm.deal(user, 1 ether);
