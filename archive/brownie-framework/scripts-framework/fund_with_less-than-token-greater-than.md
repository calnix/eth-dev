# fund\_with\_\<token>()

In the example of Lottery.sol, we had to fund Lottery.sol with LINK tokens so that it could call VRF Coordinator for randomness and pay the required oracle gas fee.

{% tabs %}
{% tab title="deploy.py" %}
```python
def end_lottery():
    account = get_account()
    lottery = Lottery[-1]
    
    # need to fund link before we end, to get randonmess to select winner
    tx = fund_with_link(contract_address=lottery.address)
    tx.wait(1)
    
    end_tx = lottery.endLottery({"from":account})
    end_tx.wait(1)
    print(".....Lottery is Ended!.....")
    time.sleep(60)                      #wait for VRF coord to callback
    print(f"Recent winner is {lottery.recentWinner()}")

def main():
    deploy()
    start_lottery()
    enter_lottery()
    end_lottery()
```
{% endtab %}

{% tab title="helpful_scripts.py" %}
```python

def fund_with_link(contract_address, account=None, link_token=None, amount=100000000000000000): #0.1 LINK
    account = account if account else get_account()     #account = account, if parameter was specified. else get_account()
    link_token = link_token if link_token else get_contract("link_token")
    
    ## send link to lottery contract, from our account
    tx = link_token.transfer(contract_address, amount, {"from": account})
    #link_token_contract = interface.LinkTokenInterface(link_token.address)
    #tx = link_token_contract.transfer(contract_address, amount, {"from": account})
    
    tx.wait(1)
    print(".....Contract Funded.....")
    return tx
```

* To transfer link to our contract, we must call transfer() from Link Token contract.&#x20;
* Token contracts are ledgers that track token balances across addresses.&#x20;
{% endtab %}
{% endtabs %}

## Ways to interact with deployed contracts&#x20;

To interact with a contract, we need its address and ABI.

In the context of a deployed contract, we would be able to get its address easily enough, leaving its ABI.

Using brownie, there are two methods:

**1. Create from contract's ABI:**\
`contract =`` `**`Contract.from_abi`**`(contract_type.name, contract_address,`` `**`contract_type.abi`**`)`

> _creates Contract object from abi and address._

**2. Get ABI from interface:**\
link\_token\_contract = interface.LinkTokenInterface(link\_token.address)\
tx = link\_token\_contract.transfer(contract\_address, amount, {"from": account})

The interface outlines all the function definitions necessary to interact with said contract -> therefore it will compile down to the same ABI as the contract.

### Explanation

Smart contracts written in high-level languages like [Solidity](https://docs.soliditylang.org/en/v0.8.2/) or [Vyper](https://vyper.readthedocs.io/en/stable/) need to be compiled in EVM executable bytecode; when a smart contract is deployed, this bytecode is stored on the blockchain and is associated with an address.

To the EVM, a smart contract is just this sequence of bytecode. To access functions defined in high-level languages, users need to translate names and arguments into byte representations for bytecode to work with it.

To interpret the bytes sent in response, users need to convert back to the tuple of return values as  defined in higher-level languages.

Languages that compile for the EVM maintain strict conventions about these conversions, but in order to perform them, one must know the precise names and types associated with the operations.

**Hence, the need for the ABI** -> it defined the function names, parameters they each take and what they return.

{% hint style="info" %}
**Compiled Contract**: The contract converted to byte-code to run on the Ethereum Virtual Machine (EVM), adhering to the [specification](https://github.com/ethereum/yellowpaper). Note the function names and input parameters are **hashed during compilation**. Therefore, for another account to call a function, it must first be given the function name and arguments - hence the ABI.



**Application Binary Interface - ABI:** A list of the contract's functions and arguments (in JSON1 format). An account wishing to use a smart contract's function uses the ABI to hash the function definition, so it can create the EVM bytecode required to call the function. This is then included in the data field, Td, of a transaction and interpreted by the EVM with the code at the target account (the address of the contract).

\
[https://ethereum.stackexchange.com/questions/234/what-is-an-abi-and-why-is-it-needed-to-interact-with-contracts](https://ethereum.stackexchange.com/questions/234/what-is-an-abi-and-why-is-it-needed-to-interact-with-contracts)
{% endhint %}

ABI defines the methods and structures used to interact with the binary contract. The ABI indicates the caller of the function to encode the needed information like function signatures and variable declarations in a format that the EVM can understand to call that function in bytecode; this is called ABI encoding.

#### TLDR:

Functions are hashed on compilation and stored as such on the EVM.&#x20;

When we want to call a function on a deployed contract, we need to send a message call that will communicate to the EVM that we are calling a specific function hash at a specific address.

The ABI lists all the available functions and methods of the contract. We use it to generate the correct function hash and send out our message call as bytecode.

The conversion from higher-level, human readable languages to lower-level bytecode is handled by the compiler.

Just provide address and abi.

[https://www.quicknode.com/guides/solidity/what-is-an-abi](https://www.quicknode.com/guides/solidity/what-is-an-abi)
