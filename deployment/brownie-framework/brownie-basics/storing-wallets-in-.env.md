# storing wallets in .env

We can store our private keys in a .env file allowing our code to reference them as variables for quick testing and easy switching between wallets.

### create .env file&#x20;

create .env file in project folder. input as follows, the name of the variable is up to you to define (here it is PRIVATE\_KEY)

![](<../../../.gitbook/assets/image (278).png>)

### create brownie-config.yaml

![](<../../../.gitbook/assets/image (3).png>)

* dotenv: .env
  * tells brownie which file to look at for pulling env variables, when running scripts
* wallets variables
  * stores list of wallets, each one pulling a different PK from env. ${PK}

We will reference wallets like so:

![](<../../../.gitbook/assets/image (257).png>)

