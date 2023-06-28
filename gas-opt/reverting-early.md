# reverting early

* Txn that revert still have to pay gas
* If they run out of gas, they pay the full limit
* If they hit a revert opcode, e.g. from a require statement, they pay for gas up till the revert opcode was hit. They do not pay for any gas afterward.
* Implication: revert as early as possible in the execution to save the user on gas
