# Brownie Networks

## Networks created

{% hint style="info" %}
> brownie networks list true

will return full details of each network
{% endhint %}

### Under Ethereum category

![](<../../.gitbook/assets/image (243).png>)

* **ganache-local is our persistent mock network**
* requires that ganacheUI is running, so that brownie will latch onto it automatically to deploy required mock contracts.
* Persistency will last till ganacheUI is closed.

{% content-ref url="brownie-advanced/forking-and-mocking.md" %}
[forking-and-mocking.md](brownie-advanced/forking-and-mocking.md)
{% endcontent-ref %}

{% content-ref url="brownie-advanced/mocking.md" %}
[mocking.md](brownie-advanced/mocking.md)
{% endcontent-ref %}

### Development:

![](<../../.gitbook/assets/image (101).png>)

#### mainnet-fork-dev

* **Forks ETH mainnet**
* Provider: Alchemy

\-- brownie networks add development mainnet-fork-dev cmd=ganache-cli host=http://127.0.0.1 fork=https://eth-mainnet.alchemyapi.io/v2/j13raIIaPyTnV9ZVv2UnQm48PUFcPDkL accounts=10 mnemonic=brownie port=8545

#### mainnet-fork

* brownie comes preset with mainnet-fork under Development category:
* connects to infura, and requires infura token in .env.
* Infura causes problems, so Patrick overwrites it to use Alchemy
* we shall do this too

brownie networks add development mainnet-fork cmd=ganache-cli host=http://127.0.0.1 fork=https://eth-mainnet.alchemyapi.io/v2/QQSu6nAAnOvWlkS9KTHk9UGk-kxieDmj accounts=10 mnemonic=brownie port=8545

## On Ganache UI and CLI

perhaps changing ganacheUI port number to be something else, could prevent brownie(ganache-cli) to auto-attach itself, if ui is running.&#x20;

This way we can control networks with port numbers.

[https://stackoverflow.com/questions/69818605/brownie-doesnt-automatically-attach-to-local-ganache-when-i-have-ganache-open-i](https://stackoverflow.com/questions/69818605/brownie-doesnt-automatically-attach-to-local-ganache-when-i-have-ganache-open-i)
