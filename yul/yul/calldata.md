# calldata

* [https://betterprogramming.pub/solidity-tutorial-all-about-calldata-aebbe998a5fc](https://betterprogramming.pub/solidity-tutorial-all-about-calldata-aebbe998a5fc)
*   `offset` is the position in the calldata where the actual data of the array starts, i.e. where the first array element is located. Let's take an example:


* [https://medium.com/swlh/getting-deep-into-evm-how-ethereum-works-backstage-ab6ad9c0d0bf](https://medium.com/swlh/getting-deep-into-evm-how-ethereum-works-backstage-ab6ad9c0d0bf)
* [https://betterprogramming.pub/solidity-tutorial-all-about-calldata-aebbe998a5fc](https://betterprogramming.pub/solidity-tutorial-all-about-calldata-aebbe998a5fc)
* [https://medium.com/@kalexotsu/understanding-solidity-assembly-hashing-a-string-from-calldata-fbd2ece82263](https://medium.com/@kalexotsu/understanding-solidity-assembly-hashing-a-string-from-calldata-fbd2ece82263)

### `calldataload`&#x20;

* `calldataload` is the EVM opcode for getting 32 bytes from `calldata`.
* The parameter to `calldataload` is an offset: typically the first 4 bytes of `calldata` is a function selector, so `calldataload(4)` is used to get the 32 bytes starting from the fifth byte&#x20;
* [https://ethereum.stackexchange.com/questions/77475/what-data-is-in-calldataload](https://ethereum.stackexchange.com/questions/77475/what-data-is-in-calldataload)

