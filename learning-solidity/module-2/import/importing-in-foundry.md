# Importing in Foundry

* Foundry is able to grab to download directly from github.

### **Example**

You want to use the IERC20.sol interface provided by Yield at: [https://github.com/yieldprotocol/yield-utils-v2/blob/main/contracts/token/IERC20.sol](https://github.com/yieldprotocol/yield-utils-v2/blob/main/contracts/token/IERC20.sol)

### **Get dependencies**

#### **First we need to download the repo into our dependencies folder (lib):**

```bash
forge install yieldprotocol/yield-utils-v2
```

![organization/repo](<../../../.gitbook/assets/image (327).png>)

Forge will install the repo into our project directory (repo can be found inside lib)

![lib](<../../../.gitbook/assets/image (66).png>)

.**gitmodules** will be updated to reflect the dependencies:

![.gitignore](<../../../.gitbook/assets/image (46).png>)

#### Run forge remappings to check how forge is mapping out your dependencies

> forge remappings

![](<../../../.gitbook/assets/image (305).png>)

`yield-utils-v2/` is mapped to directory `lib/yield-utils-v2/contracts`

**This informs us how to craft our import statement.**

### Craft import statement&#x20;

```solidity
// points to lib/yield-utils-v2/contracts/token/IERC20.sol
import "yield-utils-v2/token/IERC20.sol";
```

![](<../../../.gitbook/assets/image (186).png>)

* Now when **`forge build`** is run, there should not be any issues.
