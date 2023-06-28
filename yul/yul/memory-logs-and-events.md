# Memory: Logs and Events

### Logs

```solidity
contract Log {
    event SomeLog(uint256 indexed a, uint256 indexed b);
    event SomeLogV2(uint256 indexed a, bool);

    function emitLog() external {
        emit SomeLog(5, 6);
    }

    function yulEmitLog() external {
        assembly {
            // keccak256("SomeLog(uint256,uint256)")
            let
                signature
            := 0xc200138117cf199dd335a2c6079a6e1be01e6592b6a76d4b5fc31b169df819cc
            log3(0, 0, signature, 5, 6)
        }
    }

```

* emitLog and yulEmitLog achieve the same outcome: emitting 5 and 6
* the topic is the signature -> hash of event and its params
* the 0 0 is reference to memory -> here we are not using any non-indexed args

### Indexed and Non-Indexed

