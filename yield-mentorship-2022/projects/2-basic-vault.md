---
description: https://github.com/yieldprotocol/mentorship2022/issues/2
---

# #2 Basic Vault

#### Problem statement: [https://github.com/calnix/Basic-Vault](https://github.com/calnix/Basic-Vault)

#### Github: [https://github.com/calnix/Basic-Vault](https://github.com/calnix/Basic-Vault)

## Objectives: multi-user safe

1. This is a single-token Vault that holds a pre-specified erc-20 token.
2. Users can send tokens to the Vault contract.
3. Vault contract records the user's token deposits.
4. Users can withdraw their tokens only to the address they deposited from.

## Contracts

* ERC20Mock.sol
* BasicVault.sol

### Vault contract

1. Deposit: After approval (handles by front-end), token is deposited via transferFrom.
2. Withdraw: Token is returned, and the Vault updates its user records.

> Vault contract should import the IERC20 interface, take the Token address in the Vault constructor, and cast it into an IERC20 state variable

### ERC2Mock contract

1. Public, unrestricted mint() function (anyone should be able to mint)
2. Use Yield's version: `import https://github.com/yieldprotocol/yield-utils-v2/blob/main/contracts/mocks/ERC20Mock.sol`

{% hint style="info" %}
To better understand the ERC20.sol contract and its capabilities see: \
[https://calnix.gitbook.io/solidity-lr/reference-contracts/erc-20.sol](https://calnix.gitbook.io/solidity-lr/reference-contracts/erc-20.sol)\
\
TLDR on using transfer vs transferFrom:\
[https://calnix.gitbook.io/solidity-lr/reference-contracts/erc-20.sol/tldr-transfer-vs-transferfrom](https://calnix.gitbook.io/solidity-lr/reference-contracts/erc-20.sol/tldr-transfer-vs-transferfrom)
{% endhint %}

## Additional Details

1. **Add full NatSpec**

* contracts should have @title, @notice, @dev, @author.
* functions should have @notice, @dev, @param, @return.
* events should have @notice (no need param or return).&#x20;

**2. Implement CI so when you push to Github, your tests are run automatically.**&#x20;

* https://gist.github.com/clifton/b5ee5286bb229281fb31d7c4b15e6f31&#x20;
* https://book.getfoundry.sh/config/continous-integration.html

## &#x20;Code

{% tabs %}
{% tab title="BasicVault" %}
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "yield-utils-v2/contracts/token/IERC20.sol";

/**
@title An on-chain name registry
@author Calnix
@dev Vault for a specific ERC20 token; token address to be passed to constructor.
@notice Vault for a pre-defined ERC20 token that was set on deployment. 
*/

contract BasicVault {
    
    ///@notice ERC20 interface specifying token contract functions
    ///@dev For constant variables, the value has to be fixed at compile-time, while for immutable, it can still be assigned at construction time.
    IERC20 public immutable wmdToken;    

    ///@notice mapping addresses to their respective token balances
    mapping(address => uint) public balances;

    /// @notice Emit event when tokens are deposited into Vault
    event Deposited(address indexed from, uint amount);
    
    /// @notice Emit event when tokens are withdrawn from Vault
    event Withdrawal(address indexed from, uint amount);
    
    ///@param wmdToken_ ERC20 contract address
    constructor(IERC20 wmdToken_){
        wmdToken = wmdToken_;
    }

    /// @notice User can deposit tokens into Vault
    /// @dev Expect Deposit to revert if transferFrom fails
    /// @param amount The amount of tokens to deposit
    function deposit(uint amount) external {    
        balances[msg.sender] += amount;

        bool success = wmdToken.transferFrom(msg.sender, address(this), amount);
        require(success, "Deposit failed!"); 
        emit Deposited(msg.sender, amount);
    }

    /// @notice User can withdraw tokens from Vault
    /// @dev Expect Withdraw to revert if transfer fails
    /// @param amount The amount of tokens to withdraw
    function withdraw(uint amount) external {      
        balances[msg.sender] -= amount;
        
        bool success = wmdToken.transfer(msg.sender, amount);
        require(success, "Withdrawal failed!");
        emit Withdrawal(msg.sender, amount);
    }
}
```
{% endtab %}

{% tab title="WMDToken" %}
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "yield-utils-v2/contracts/mocks/ERC20Mock.sol";

contract WMDToken is ERC20Mock {
    
    ///@dev Inherited constructor from ERC20Mock.sol. 
    ///@dev No parameters need to be passed for top-level constructor.
    constructor() ERC20Mock("WoMiGawd", "WMD") {

    }
}
```
{% endtab %}
{% endtabs %}

### Checks-effect-interaction pattern

<mark style="color:red;">**Why does it matter?**</mark>

When calling an external address, for example when transferring tokens to another account, the calling contract is also transferring the control flow to the external entity.&#x20;

Assuming this external entity is a smart contract as well, the <mark style="background-color:yellow;">**external entity is now in charge of the control flow**</mark> and can execute any inherent code within it.

**This can leave your contract open to re-entrancy attacks.**&#x20;

The high-level idea is as follows:

1. check & update the internal states (balances)
2. keep external interactions (function calls) to the last step

{% hint style="warning" %}
Essentially we will update our mappings balances to reflect any outflow, before initiating the transfer.

Further reading:\
[https://www.securing.pl/en/reentrancy-attack-in-smart-contracts-is-it-still-a-problem/](https://www.securing.pl/en/reentrancy-attack-in-smart-contracts-is-it-still-a-problem/)
{% endhint %}

## Testing

* Vault contract should be fully tested.
* No need to test ERC20Mock. Assume it has been battle-tested.
* WMDTokenWithFailedTransfers.sol is used to simulate token transfer failure, so that we can ensure our Vault functions behave accordingly in such situations. &#x20;

### Testing States

**StateZero** (user has no tokens)

* cannot deposit `(testUserCannotWithdraw)`
* cannot withdraw `(testUserCannotDeposit)`
* fuzz testing `(testUserMintApproveDeposit)`&#x20;

**StateTokensMinted** (user has minted 100 wmd tokens - only action available is deposit)

* if transfer fails, deposit should revert `(testDepositRevertsIfTransferFails)`
* deposit tokens into vault `(testDeposit)`

**StateTokensDeposited** (user has deposited tokens into Vault)

* cannot withdraw if transfer fails `(testWithdrawRevertsIfTransferFails)`
* cannot withdraw more than deposit `(testUserCannotWithdrawExcessOfDeposit)`
* partial withdrawal `(testUserWithdrawPartial)`
* full withdrawal `(testUserWithdrawAll)`

{% hint style="warning" %}
It’s better to test for a specific error type, than use `vm.expectRevert` as a catch-all leading to false positives.

As such, with regards to failing token transfers, observe the use of `stdError.arithmeticError` in both&#x20;

1. testFuzzUserCannotWithdraw
2. testUserCannotWithdrawExcessOfDeposit



[https://book.getfoundry.sh/reference/forge-std/arithmeticError.html](https://book.getfoundry.sh/reference/forge-std/arithmeticError.html)
{% endhint %}

#### Testing Guidelines

* All state variable changes in the contracts that you code.
* All state variable changes in other contracts caused by calls from contracts that you code.
* All require or revert in the contracts that you code.
* All events being emitted.
* All return values in contracts that you code.

{% tabs %}
{% tab title="BasicVaultTest.t.sol" %}
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import "forge-std/console2.sol";

import 'src/BasicVault.sol';
import 'test/WMDTokenWithFailedTransfers.sol';

abstract contract StateZero is Test {
        
    BasicVault public vault;
    WMDTokenWithFailedTransfers public wmd;
    address user;

    event Deposited(address indexed from, uint amount);
    event Withdrawal(address indexed from, uint amount);

    function setUp() public virtual {
        wmd = new WMDTokenWithFailedTransfers();
        vault = new BasicVault(wmd);

        user = address(1);
        vm.label(user, "user");
    }
}

contract StateZeroTest is StateZero {
    
    function testGetUserBalance() public {
        console2.log("Check User has zero tokens in vault");
        vm.prank(user);
        uint balance = vault.balances(user);
        assertTrue(balance == 0);
    }

    function testUserCannotDeposit(uint amount) public {
        console2.log("User cannot deposit without tokens");
        vm.assume(amount > 0);
        vm.prank(user);
        vm.expectRevert("ERC20: Insufficient approval");
        vault.deposit(amount);
    }
   
    function testUserMintApproveDeposit(uint amount) public {
        console2.log("User mints tokens and deposits into vault");
        vm.startPrank(user);
        vm.assume(amount > 0);
        wmd.mint(user, amount);
        wmd.approve(address(vault), amount);

        vm.expectEmit(true, false, false, true);
        emit Deposited(user, amount);
        vault.deposit(amount);

        assertTrue(wmd.balanceOf(user) == 0);
        assertTrue(vault.balances(user) == amount);
        vm.stopPrank();
    }

}

abstract contract StateMinted is StateZero {
    uint userTokens;
    
    function setUp() public override virtual {
        super.setUp();
        
        // state transition: user mints 100 tokens
        userTokens = 100 * 10**18;
        vm.prank(user);
        wmd.mint(user, userTokens);
    }
}

contract StateMintedTest is StateMinted {

    function testFuzzUserCannotWithdraw(uint amount) public {
        console2.log("User cannot withdraw with no balance");
        vm.assume(amount > 0 && amount < wmd.balanceOf(user));
        vm.prank(user);
        vm.expectRevert(stdError.arithmeticError);
        vault.withdraw(amount);
    }

    function testDepositRevertsIfTransferFails() public {
        console2.log("Deposit transaction should revert if transfer fails");
        wmd.setFailTransfers(true);
        
        vm.startPrank(user);
        wmd.approve(address(vault), userTokens);
        vm.expectRevert("Deposit failed!");
        vault.deposit(userTokens);
        vm.stopPrank();
    }

    function testDeposit() public {
        console2.log("User deposits into Vault");
        vm.startPrank(user);
        wmd.approve(address(vault), userTokens);
        
        vm.expectEmit(true, false, false, true);
        emit Deposited(user, userTokens);
        vault.deposit(userTokens);
        
        assertTrue(vault.balances(user) == userTokens);
        assertTrue(wmd.balanceOf(user) == 0);
        vm.stopPrank();
    }

}

abstract contract StateDeposited is StateMinted {
    
    function setUp() public override virtual {
        super.setUp();  
        vm.startPrank(user);
        wmd.approve(address(vault), userTokens);
        vault.deposit(userTokens);
        vm.stopPrank();
    }
}

contract StateDepositedTest is StateDeposited {
    
    function testWithdrawRevertsIfTransferFails() public {
        console2.log("Withdraw transaction should revert if transfer fails");
        wmd.setFailTransfers(true);
        vm.prank(user);
        vm.expectRevert("Withdrawal failed!");
        vault.withdraw(userTokens);
    }

    function testUserCannotWithdrawExcessOfDeposit() public {
        console2.log("User cannot withdraw more than he has deposited");
        vm.prank(user);
        vm.expectRevert(stdError.arithmeticError);
        vault.withdraw(userTokens + 100*10**18);
    }

    function testUserWithdrawPartial() public {
        console2.log("User withdraws half of deposit from Vault");
        vm.prank(user);

        vm.expectEmit(true, false, false, true);
        emit Withdrawal(user, userTokens/2);
        vault.withdraw(userTokens/2);

        assertEq(vault.balances(user), userTokens/2);
        assertEq(wmd.balanceOf(user), userTokens/2);
    }
 
    function testUserWithdrawAll() public {
        console2.log("User to withdraw all deposits from Vault");
        vm.prank(user);

        vm.expectEmit(true, false, false, true);
        emit Withdrawal(user, userTokens);
        vault.withdraw(userTokens);

        assertEq(vault.balances(user), 0);
        assertEq(wmd.balanceOf(user), userTokens);
    }
}
```
{% endtab %}

{% tab title="WMDTokenWithFailedTransfers.sol" %}
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import 'src/WMDToken.sol';


contract WMDTokenWithFailedTransfers is WMDToken {

    bool public transferFail = false;

    function setFailTransfers(bool state_) public {
        transferFail = state_;
    }

    function _transfer(address src, address dst, uint wad) internal override returns (bool) {
        if (transferFail) {
            return false;
        } 
        else {
            return super._transfer(src, dst, wad);
        }
    }
}
```
{% endtab %}
{% endtabs %}

{% hint style="success" %}
I realized that for future testing perhaps splitting each state into a different file would be more sensible and accessible for the reader.&#x20;
{% endhint %}

## Github CI

{% code title="foundry-ci.yml" %}
```yaml
name: Foundry-CI

on: 
  push:
    branches:
      - RevisedCode
  pull_request:

jobs:
  run-tests:                    # job_id value
    name: Basic Vault           # Use jobs.<job_id>.name to a name for the job, which is displayed on GitHub.
    runs-on: ubuntu-latest     # Container OS env
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: recursive

      - name: Install Foundry
        uses: onbjerg/foundry-toolchain@v1
        with:
          version: nightly

      - name: Run Forge tests
        run: forge test
        id: test
```
{% endcode %}

* Its interesting to note that I could not apply this with windows-latest container. The action Install Foundry will return an <mark style="color:red;">Error: Unexpected HTTP response: 404)</mark>

## Deployment

On Rinkeby&#x20;

* WMD Token: https://rinkeby.etherscan.io/address/0x944403ee436a6dff974983a2fa84ff37c587bad1#writeContract
* Vault: https://rinkeby.etherscan.io/address/0xc5a93d9c0337352f0c2fd2743a2ffffd69818486#writeContract

{% tabs %}
{% tab title="MakeFile" %}
```bash
# include .env file and export its env vars (-include to ignore error if it does not exist)
-include .env

deploy-wmd:
	forge create src/WMDToken.sol:WMDToken --private-key ${PRIVATE_KEY_EDGE} --rpc-url ${RINKEBY_RPC_URL}

verify-wmd:
	forge verify-contract --chain-id ${RINKEBY_CHAINID} --compiler-version v0.8.13+commit.abaa5c0e ${WMD_CONTRACT_ADDRESS} src/WMDToken.sol:WMDToken ${ETHERSCAN_API_KEY} --num-of-optimizations 200 --flatten
	
verify-check-wmd:
	forge verify-check --chain-id ${RINKEBY_CHAINID} ${WMD_GUID} ${ETHERSCAN_API_KEY}

deploy-vault:
	forge create src/BasicVault.sol:BasicVault --private-key ${PRIVATE_KEY_EDGE} --rpc-url ${ETH_RPC_URL} --constructor-args "0xB725d02bf6B89E659762A4760109a8478C4d22D0"

verify-vault:
	forge verify-contract --chain-id ${RINKEBY_CHAINID} --compiler-version v0.8.13+commit.abaa5c0e ${VAULT_CONTRACT_ADDRESS} src/BasicVault.sol:BasicVault ${ETHERSCAN_API_KEY} --num-of-optimizations 200 --flatten --constructor-args 0x000000000000000000000000b725d02bf6b89e659762a4760109a8478c4d22d0

verify-check-vault:
	forge verify-check --chain-id ${RINKEBY_CHAINID} ${VAULT_GUID} ${ETHERSCAN_API_KEY}
```
{% endtab %}

{% tab title=".env" %}
```bash
export PRIVATE_KEY_EDGE = <your private key>
export ETHERSCAN_API_KEY = <your api key>

//Chain info
export KOVAN_CHAINID = 42
export KOVAN_RPC_URL = <input the https address infura provides you>

export RINKEBY_CHAINID = 4
export RINKEBY_RPC_URL = <input the https address infura provides you>


// Token Deployment
export WMD_CONTRACT = src/WMDToken.sol:WMDToken
export WMD_CONTRACT_ADDRESS = 0x944403ee436a6dff974983a2fa84ff37c587bad1
export WMD_GUID = 62h8tg2r61vcusbzcva8q53bus2ixmyr7apkt24hk3uzwai7nz

// Vault Deployment
export VAULT_CONTRACT = src/BasicVault.sol:BasicVault
export VAULT_CONTRACT_ADDRESS = 0xc5a93d9c0337352f0c2fd2743a2ffffd69818486
export VAULT_GUID = t4ffir6s3mmqudkr9cjutjiscj2hq39pqttteenrcpkgv9jnis
```
{% endtab %}
{% endtabs %}

As previously mentioned, the values of contract address and GUID have to be updated in .env as we flow through the make commands.

#### When deploying the Vault contract, we need to pass the WMDToken contract address together with the flag, like so:

```solidity
forge create src/BasicVault.sol:BasicVault --private-key <0x123214> --rpc-url https://rinkeby.infura.io/v3/8f4d69691d4a4636acb00ec3c933291b --constructor-args 0x944403EE436a6dFF974983a2fA84FF37c587BAD1
```

#### Subsequently, when verifying we need to pass the abi-encoded constructor arguments together with the --constructor-args flag

```solidity
cast abi-encode "constructor(address)" 0x944403EE436a6dFF974983a2fA84FF37c587BAD1

//Output:
0x000000000000000000000000944403ee436a6dff974983a2fa84ff37c587bad1
```

Take the output from above and pass it into your verify command, like so:

```solidity
forge verify-contract --chain-id 4 --compiler-version v0.8.13+commit.abaa5c0e 0xc5a93d9c0337352f0c2fd2743a2ffffd69818486 src/BasicVault.sol:BasicVault I5CK8CJ7FZ6BWQ6YGFD4U3E7CFTNWQJ3A3 --num-of-optimizations 200 --flatten --constructor-args 0x000000000000000000000000944403ee436a6dff974983a2fa84ff37c587bad1
```
