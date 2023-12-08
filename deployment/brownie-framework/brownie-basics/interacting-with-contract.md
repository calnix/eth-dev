# Interacting with contract

{% tabs %}
{% tab title="deploy.py" %}
```python
#brownie has an accounts package
from brownie import accounts, config, SimpleStorage

#put all the deplyment logic in one fucntion:
def deploy_simple_storage():
    
    #local ganache accounts
    account = accounts[0]

    #import contract into script: from brownie import <name of contract>
    simple_storage = SimpleStorage.deploy({"from": account})

    #interact with smart contract, like remix
    stored_value = simple_storage.retrieve()  #view function, no need transaction - no need account
    print(stored_value)

    #updating state
    txn = simple_storage.store(3, {"from":account})
    txn.wait(1)
    updated_stored_value = simple_storage.retrieve()
    print(updated_stored_value)
    

def main():
    deploy_simple_storage()
```
{% endtab %}

{% tab title="SimpleStprage.sol" %}
```solidity
contract SimpleStorage{

uint public favNum;

struct People {
    uint favNum;
    string name;
}

/*
declare person has People type. (like address, uint)
assign people values of 2 & Patrick  --> People({})
this is the same as --> uint public favNum = 5;
    People public person = People({favNum: 2, name: "Patrick"});
*/

People[] public people;  
mapping(string => uint) public name2num;

function store(uint _num) public {
    favNum = _num;
}

function retrieve() public view returns(uint){
    return favNum;
}

function addPerson(uint _favnum, string memory _name) public {
    people.push(People({favNum: _favnum,name: _name}));
    name2num[_name] = _favnum;
}

}
```
{% endtab %}
{% endtabs %}

### retrieve: view function

```python
    stored_value = simple_storage.retrieve()  
    print(stored_value)
```

retrieve is a view function in SimpleStorage.sol. view functions operate on calls, not transactions, so we do not need to define an account here.

### store: mutable function

```python
   #updating state
    txn = simple_storage.store(3, {"from":account})
    txn.wait(1)
    
    updated_stored_value = simple_storage.retrieve()
    print(updated_stored_value)
```

calling store requires a transaction and gas to be paid, so we need to define the account. the 3 is passed into uint favNum.

txn.wait is important for "mining".

![](<../../../.gitbook/assets/image (232).png>)

