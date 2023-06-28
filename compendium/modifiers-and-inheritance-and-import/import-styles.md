---
description: https://betterprogramming.pub/solidity-tutorial-all-about-imports-c65110e41f3a
---

# import styles

## Global Import

The statement below will import all _**Solidity objects**_ found in `“./MySolidityFile.sol”`

```
import “./MySolidityFile.sol”;
```

> &#x20;_“Solidity object”_ to describe any `contract`, `library`, `interface`, or other `constant` or variables that you can define at the file level (`struct`, `enum`, etc.)

### Instead, the Solidity docs recommend specifying imported symbols explicitly.

```solidity
Import { Something } from “./MySolidityFile.sol”;
```

* You can then mention within the curly braces `{ }` the specific symbols/objects that you want to import and use

## Import Aliases

### Global Aliases

```solidity
// longer syntax
import * as Endpoints from “./Endpoints.sol”.

// shorter syntax
import “./Endpoints.sol” as Endpoints;
```

### **Specific Aliases** <a href="#f48f" id="f48f"></a>

```solidity
import { Point as Coordinate, GPS } from “./Endpoints.sol”;
```

