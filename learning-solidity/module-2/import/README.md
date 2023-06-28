# Import

## Global Import

The statement below will import all _**Solidity objects**_ found in `“./MySolidityFile.sol”`

```solidity
import “./MySolidityFile.sol”;
```

* MySolidityFile.sol could contain multiple contracts; all of them will be imported.&#x20;

> &#x20;_“Solidity object”_ to describe any `contract`, `library`, `interface`, or other `constant` or variables that you can define at the file level (`struct`, `enum`, etc.)

#### Instead, the Solidity docs recommend specifying imported symbols explicitly.

```solidity
Import { ContractA } from “./MySolidityFile.sol”;
```

* You can specify within the curly braces `{ }` the specific symbols/objects that you want to import and use.

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

