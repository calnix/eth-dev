# Overview

## forge init&#x20;

> forge init \<project\_name>

* creates a new directory named after your project from the default template.&#x20;
* also initializes a new `git` repository.

#### Creating a New Project from template

> &#x20;forge init --template https://github.com/foundry-rs/forge-template \<project\_name>

#### Project Layout

You should see 3 directories:

* lib -> dependencies are stored as git submodules
* src -> default directory for contracts
* test -> default directory for tests

![](<../.gitbook/assets/image (236).png>)

You can configure where Forge looks for both dependencies and contracts using the `--lib-paths` and `--contracts` flags respectively. Alternatively you can configure it in `foundry.toml`

{% hint style="info" %}
For automatic Hardhat support you can also pass the `--hh` flag, which sets the following flags: `--lib-paths node_modules --contracts contracts.`
{% endhint %}

## forge build

* compiles project

## forge test

* run all tests
* creates two new directories
  * out -> contains contract artifacts, such as the ABI
  * cache -> used by `forge` to only recompile what is necessary
