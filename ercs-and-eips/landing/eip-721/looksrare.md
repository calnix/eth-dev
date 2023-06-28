# LooksRare

**sellorderDigest() is hash() in:** https://github.com/LooksRare/contracts-exchange-v1/blob/master/contracts/libraries/OrderTypes.sol

hash is used to generate the order digest:

```
bytes32 digest = keccak256(abi.encodePacked("\x19\x01", DOMAIN_SEPARATOR, makerOrder.hash()));
```

#### :: invalid S parameter&#x20;

* https://ethereum.stackexchange.com/questions/83174/is-it-best-practice-to-check-signature-malleability-in-ecrecover&#x20;
* https://crypto.iacr.org/2019/affevents/wac/medias/Heninger-BiasedNonceSense.pdf&#x20;
* from(https://github.com/LooksRare/contracts-exchange-v1/blob/master/contracts/libraries/SignatureChecker.sol)

#### :: buy()

* checkValiditySignature() --> https://github.com/LooksRare/contracts-exchange-v1/blob/master/contracts/orderValidation/OrderValidatorV1B.sol
* \_validateEOA() --> https://github.com/LooksRare/contracts-exchange-v1/blob/master/contracts/orderValidation/OrderValidatorV1B.sol

#### ::sign msg&#x20;

* https://github.com/LooksRare/contracts-exchange-v1/blob/c16227781294f23241f80c4bcc3adf77aaa81157/contracts/orderValidation/OrderValidatorV1B.sol#L438
* https://medium.com/@angellopozo/ethereum-signing-and-validating-13a2d7cb0ee3

#### :: order hash&#x20;

* https://looksrare.dev/reference/orders-schema&#x20;
* https://github.com/LooksRare/contracts-exchange-v1/blob/master/contracts/libraries/OrderTypes.sol

#### ecrecover&#x20;

* https://soliditydeveloper.com/ecrecover

#### eip721&#x20;

* https://medium.com/metamask/eip712-is-coming-what-to-expect-and-how-to-use-it-bb92fd1a7a26&#x20;
* https://medium.com/mycrypto/the-magic-of-digital-signatures-on-ethereum-98fe184dc9c7
