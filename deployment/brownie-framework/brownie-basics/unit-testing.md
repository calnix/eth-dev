# Unit Testing

* create python test files in tests folder.
* filenames must begin with test\_
* from brownie import \<contract\_name>, accounts

In this following example, we want to test that favNum initializes to 0.

* deploy the contract
* retrieve the starting value
* compare it with expected starting value

```python
from brownie import SimpleStorage, accounts

def test_deploy():
    # Arrange - setup
    account = accounts[0]

    # Act
    simple_storage = SimpleStorage.deploy({"from": account})
    starting_value = simple_storage.retrieve()
    expected_value = 0

    # Assert
    assert expected_value == starting_value

```

To run this test, input: `brownie test` in terminal:

![](<../../../.gitbook/assets/image (254).png>)

If we get a green dot at the end like above, all tests were successful.&#x20;

### Test updating storage variable

```python
from brownie import SimpleStorage, accounts

def test_deploy():
    # Arrange - setup
    account = accounts[0]

    # Act
    simple_storage = SimpleStorage.deploy({"from": account})
    starting_value = simple_storage.retrieve()
    expected_value = 0

    # Assert
    assert expected_value == starting_value

def test_update_storage():
    # Arrange - setup
    account = accounts[0]
    simple_storage = SimpleStorage.deploy({"from": account})

    # Act
    expected = 15
    simple_storage.store(expected,{"from": account}) #store 15

    # Assert
    assert expected == simple_storage.retrieve()

```

* deploy contract
* update favNum storage variable to be 15
* retrieve storage variable and compare with expected.

### Tips

To test 1 function

> brownie test -k \<function name>\
> brownie test -k test\__update_\_storage

To get into interactive mode:

> brownie test --pdb

when a test fails, or an error is encountered, terminal will convert to an interactive python shell:

![](<../../../.gitbook/assets/image (308).png>)

we can observe the values of variables by typing them in and their values will be returned.

{% hint style="info" %}
reference the other brownie flags from pytest documentation
{% endhint %}

## get accounts

```python
from brownie import SimpleStorage, accounts, network

#if on non-persistent dev netowrk, use generated ganache account
#else look in.yaml file for our MM account's PK
def get_accounts():
    if network.show_active() == "development":
        return accounts[0]                  #use ganache generated account.
    else:
        return accounts.add(config["wallets"]["wallet1"])  #look in config.yaml
        
def test_deploy():
    # Arrange - setup
    account = get_accounts()

#etc
```

## Negative tests

Lets say you want to do a test that results in a revert. Like testing a function that only the owner can call, but calling it from a non-owner address.&#x20;

We will use pytest to skip tests that we only want conducted in our private local chains (pip install pytest)

{% code title="test_fund_me.py" %}
```python
def test_OwnerWithdraw():
    if network.show_active() not in LOCAL_BLOCKCHAIN_ENV:
       pytest.skip("ONLY FOR LOCAL TESTING!")

    if len(FundMe) <= 0: 
        fund_me = deploy_fund_me()
    fund_me = FundMe[-1]

    bad_actor = accounts.add()
    fund_me.withdraw({"from": bad_actor})
```
{% endcode %}

You would expect a revert to be thrown, like below, reflecting that an exception had occurred:

![exception thrown](<../../../.gitbook/assets/image (297).png>)

### How can we conduct a negative test properly?

the correct form of the test would be:

```python
from brownie import exceptions

def test_only_owner_can_withdraw():
    if network.show_active() not in LOCAL_BLOCKCHAIN_ENVIRONMENTS:
        pytest.skip("only for local testing")
    fund_me = deploy_fund_me()
    bad_actor = accounts.add()
    with pytest.raises(exceptions.VirtualMachineError):
        fund_me.withdraw({"from": bad_actor})
```

## Where should i run my tests

1. Brownie Ganache chain with Mocks: Always&#x20;
   1. (tests should clear on brownie spun up ganache-cli env)
2. Testnet: Always (but only for integration testing)
3. Brownie mainnet-fork: Optional&#x20;
4. Custom mainnet-fork: Optional
5. Self/Local Ganache: Not necessary, but good for tinkering
