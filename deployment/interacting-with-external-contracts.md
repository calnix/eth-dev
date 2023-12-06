# Interacting with External Contracts

Say you want to interact with a deployed contract, but you don't have the published source code or interface to work with.

You can work with the ABI of the contract + address.

Assuming the ABI is not published, you can extract it from the website front-end:

* Dev Console -> Sources -> js bundles
* look through the js folder, the abi will be a long string in there somewhere

Once you extract the ABI:

Ensure the abi starts off in this manner: \[{....

![](<../.gitbook/assets/image (11) (1).png>)

If it starts with abi =\[.. you can drop that bit.

{% hint style="info" %}
When you compile with brownie, compiled code is placed into `build` directory as a .json file. That 10.000 lines of code is not abi, abi is the first property of that json file.
{% endhint %}

### Loading the ABI into Brownie

```python
with open("./scripts/IdleGameAbi.json") as file:
    contract_abi = json.load(file)

# contract
contract_address = "0x82a85407BD612f52577909F4A58bfC6873f14DA8"
contract = Contract.from_abi("IdleGame", contract_address, contract_abi)

```

![](<../.gitbook/assets/image (114).png>)

#### Some reference:

[https://ethereum.stackexchange.com/questions/114238/creating-a-contract-contrainer-with-brownie](https://ethereum.stackexchange.com/questions/114238/creating-a-contract-contrainer-with-brownie)
