---
description: ERC-1167
---

# Minimal Proxy Example

## Bytecode of minimal proxy

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

1. **Initialisation code: `3d602d80600a3d3981f3`**
   * This bit is responsible for deployment and stores the runtime bytecode on-chain.
   * To know more, read up on [smart contract creation code](https://www.rareskills.io/post/ethereum-contract-creation-code).
2. **Copy calldata: `363d3d373d3d3d363d73`**
   * When a transaction call is made to the minimal proxy, the proxy save the calldata passed to memory so that it can forward the said calldata to the implementation contract via `delegatecall`.
   * This is how a function call and its parameters are passed along to the main contract to be executed.
   * Note that while the execution logic is stored on the implementation contract, with the use of `delegatecall`, the execution logic is “ported over” and executed within the context of the proxy. This means that the storage of the proxy gets modified, not the implementation contract.
3. **Implementation address: `bebebebebebebebebebebebebebebebebebebebe`**
   * This is a placeholder address and should be replaced with the address of the actual implementation contract.
4. **Delegatecall instruction: `5af43d82803e903d91602b57fd5bf3`**
   * Once the delegate call is executed, the minimal proxy will return the result of the call if it was successful. If an error occurred, it will revert the transaction.

### **Deploying minimal proxies with CREATE2**

```solidity
/// @dev Deploys a new minimal contract via create2
/// @param implementation Address of Implementation contract
/// @param salt Random number of choice
function deploy(address implementation, uint256 salt) external returns (address) {
  
  // cast address to bytes
  bytes20 implementationBytes = bytes20(implementation);

  // minimal proxy address
  address proxy;

  assembly {
      
      // free memory pointer
      let pointer := mload(0x40)
  
      // mstore 32 bytes at the start of free memory 
      mstore(pointer, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)

      // overwrite the trailing 0s above with implementation contract's address in bytes
      mstore(add(pointer, 0x14), implementationBytes)
     
      // store 32 bytes to memory starting at "clone" + 40 bytes
      mstore(add(pointer, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)

      // create a new contract, send 0 Ether
      proxy := create2(0, pointer, 0x37, salt)
  }

  return proxy;
}
```

Since we need to manipulate the bytecode, to update the implementation contract address, the assembly that you see above is necessary.&#x20;

For the most part, the steps do not change and end-users can simply pass their implementation contract’s address as a parameter. Note that using `create2` requires a salt to be passed. In this context, a salt is an arbitrary value (32 bytes) of the sender’s choice.
