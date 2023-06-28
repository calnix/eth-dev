---
description: https://eips.ethereum.org/EIPS/eip-3156
---

# Implementing ERC3156

A flash lending feature integrates two smart contracts using a callback pattern. These are called the LENDER and the RECEIVER.

A flashloan lender must implement the IERC3156FlashLender interface. Hence our vault contract will inherit IERC3156FlashLender

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "lib/yield-utils-v2/contracts/mocks/ERC20Mock.sol";
import "lib/openzeppelin-contracts/contracts/access/Ownable.sol";

import "lib/yield-utils-v2/contracts/token/IERC20.sol";
import "src/IERC3156FlashLender.sol";
import "src/IERC3156FlashBorrower.sol";

/**
@title Flash Loan Server
@author Calnix
@dev Contract allows users to exchange a pre-specified ERC20 token for some other wrapped ERC20 tokens.
@notice Wrapped tokens will be burned, when user withdraws their deposited tokens.
*/

contract FlashLoanVault is ERC20Mock, IERC3156FlashLender {

    ///@dev The keccak256 hash of "ERC3156FlashBorrower.onFlashLoan"
    bytes32 public constant CALLBACK_SUCCESS = keccak256("ERC3156FlashBorrower.onFlashLoan");

    ///@notice Fee is a constant 0.1%
    ///@dev 1000 == 0.1%  (1 == 0.0001 %)
    uint256 public constant fee = 1000; 
    
    ///@dev mapping of supported addresses by Flash loan provider
    mapping(address => bool) public supportedTokens;

    ///@dev ERC20 interface specifying token contract functions
    IERC20 public immutable underlying;   
```

## Lender

The Lending contract needs to implement the following functions

### flashfee & \_flashfee()

```solidity
    //Note: The flashFee function MUST return the fee charged for a loan of amount token. If the token is not supported flashFee MUST revert.
    ///@dev The fee to be charged for a given loan.
    ///@param token The loan currency.
    ///@param amount The amount of tokens lent.
    ///@return The amount of `token` to be charged for the loan, on top of the returned principal.
    function flashFee(address token, uint256 amount) external view returns (uint256) {
        require(supportedTokens[token], "FlashLender: Unsupported currency");
        return _flashFee(token, amount);
    }


    ///@dev The fee to be charged for a given loan. Internal function with no checks.
    ///@param token The loan currency.
    ///@param amount The amount of tokens lent.
    ///@return The amount of `token` to be charged for the loan, on top of the returned principal.
    ///Note: division of 10000 is for rebasing fee from its integer to percentage form.
    function _flashFee(address token, uint256 amount) internal view returns (uint256) {
        return amount * fee / 10000;
    }
```

These two functions are used to set the fee for each token the lender is willing to lend out. In our implementation we will only be lending a single token, DAI, hence the fee calculation is static.

### maxFlashLoan()

```solidity
    ///Note: The maxFlashLoan function MUST return the maximum loan possible for token. If a token is not currently supported maxFlashLoan MUST return 0, instead of reverting.
    ///@dev The amount of currency available to be lended.
    ///@param token The loan currency.
    ///@return The amount of `token` that can be borrowed -> max of DAI deposited.
    function maxFlashLoan(address token) external view returns (uint256) {
        return supportedTokens[token] ? IERC20(token).balanceOf(address(this)) : 0;
    }
```

* Returns the maximum amount of a token the lender is able to offer in a flash loan - dependent on the lender's holdings.
* Also, it is used to tell when a token is not support (or does not have liquidity) by returning a zero.

### flashloan()

flashLoan() function executes the flash loan. \
A borrower would call on this function to execute a flash loan (via flashBorrow).&#x20;

1. The receiver address must be a contract implementing the borrower interface. Any arbitrary data may be passed in addition to the call.
2. The only requirement for the implementation details of the function are that you have to call the onFlashLoan callback from the receiver:

> require(receiver.onFlashLoan(msg.sender, token, amount, fee, data) == keccak256("ERC3156FlashBorrower.onFlashLoan"), "IERC3156: Callback failed");&#x20;

After the callback, the flashLoan function must take the amount + fee token from the receiver, **or revert if this is not successful.**

```solidity
    
    /** Note: The flashLoan function MUST include a callback to the onFlashLoan function in a IERC3156FlashBorrower contract.
     * @dev Loan `amount` tokens to `receiver`, and takes it back plus a `flashFee` after the callback.
     * @param receiver The contract receiving the tokens, needs to implement the `onFlashLoan(address user, uint256 amount, uint256 fee, bytes calldata)` interface.
     * @param token The loan currency.
     * @param amount The amount of tokens lent.
     * @param data A data parameter to be passed on to the `receiver` for any custom use.
     */
    function flashLoan(IERC3156FlashBorrower receiver, address token, uint256 amount, bytes calldata data) external returns(bool) {
        require(supportedTokens[token], "FlashLender: Unsupported currency");

        uint256 _fee = _flashFee(token, amount);
        require(IERC20(token).transfer(address(receiver), amount), "FlashLender: Transfer failed");

        require(receiver.onFlashLoan(msg.sender, token, amount, _fee, data) == CALLBACK_SUCCESS, "IERC3156: Callback failed");
        require(IERC20(token).transferFrom(address(receiver), address(this), amount + _fee), "FlashLender: Repay failed");

        return true;
    }
```

Function checks for the following:

* Tokens that was being requested for flash loan are supported
* Calculates fee to be charged for flash loan amount
* Transfers the loan amount to the receiver
* **Callback**: calls back to receiver. Receiver has to be an `IERC3156FlashBorrower` and contain the function `onFlashLoan` as described in the next section. This function ensures the legitimacy of the flash loan by
  * verifying the sender is the correct lender
  * verifying the initiator for the flash loan was actually the receiver contract&#x20;
  * returns `keccak256("ERC3156FlashBorrower.onFlashLoan")`
* Finally, the amount+fee is transferred from the receiver to the lender.

## Receiver

Receiver has to implement IERC3156FlashBorrower interface. This will allow for callback pattern.

The borrower interface consists of only of 1 callback function: onFlashLoan(). We will overwrite this in our implementation.

Lender is a fixed IERC3156FlashLender contract **defined on deployment.**

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "lib/yield-utils-v2/contracts/token/IERC20.sol";
import "src/IERC3156FlashLender.sol";
import "src/IERC3156FlashBorrower.sol";

contract FlashBorrower is IERC3156FlashBorrower {

    enum Action {NORMAL, OTHER}

    //lender is a fixed IERC3156FlashLender contract defined on deployment
    IERC3156FlashLender lender;

    constructor (IERC3156FlashLender lender_) {
        lender = lender_;
    }
```

* Flash lender address is passed as IERC3156FlashLender into the constructor.

### flashBorrow()

```solidity
    /// @notice Approve the token to the lender for the total repayment amount, then take flashloan
    /// @dev Initiate a flash loan
    function flashBorrow(address token, uint256 amount) public {
        bytes memory data = abi.encode(Action.NORMAL);

        uint256 _allowance = IERC20(token).allowance(address(this), address(lender));
        uint256 _fee = lender.flashFee(token, amount);
        uint256 _repayment = amount + _fee;

        IERC20(token).approve(address(lender), _allowance + _repayment);
        lender.flashLoan(this, token, amount, data);
    }
```

For the transaction to not revert, inside the **`onFlashLoan`** the borrower contract **must approve amount + fee of the token to be taken by msg.sender**.



### OnFlashLoan()

```solidity
function onFlashLoan(address initiator, address token, uint256 amount, uint256 fee, bytes calldata data) external override returns(bytes32) {
    require(msg.sender == address(lender), "FlashBorrower: Untrusted lender");
    require(initiator == address(this), "FlashBorrower: Untrusted loan initiator");
    
    // optionally check data here if wanted
    (Action action) = abi.decode(data, (Action));
    if (action == Action.NORMAL) {
        // do one thing
    } else if (action == Action.OTHER) {
        // do another
    }
    return keccak256("ERC3156FlashBorrower.onFlashLoan");
}
```

**This does three things:**

* verify the sender is actually the lender
* verify the initiator for the flash loan was actually our contract
* return the pre-defined hash to verify a successful flash loan

We could further implement additional logic here based on the passed data field if required. Essentially, what do we do with the flash loan once we have received it.

{% hint style="warning" %}
Borrower calls **`flashBorrow`** to initiate flash loan, which then calls **`flashLoan`** on the Lender contract.
{% endhint %}



{% tabs %}
{% tab title="FlashBorrower.sol" %}
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "lib/yield-utils-v2/contracts/token/IERC20.sol";
import "src/IERC3156FlashLender.sol";
import "src/IERC3156FlashBorrower.sol";

// https://github.com/alcueca/ERC3156/blob/main/contracts/FlashBorrower.sol
/*
We'll implement a flashBorrow function. 

lender is a fixed IERC3156FlashLender contract defined on deployment
We first approve the token to the lender for the total repayment amount. The repayment is calculated as the loan amount + fee. We can get the fee by calling flashFee on the lender.

Lastly we can execute the flashLoan function.

*/

contract FlashBorrower is IERC3156FlashBorrower {

    enum Action {NORMAL, OTHER}

    //lender is a fixed IERC3156FlashLender contract defined on deployment
    IERC3156FlashLender lender;

    constructor (IERC3156FlashLender lender_) {
        lender = lender_;
    }


    /// @notice Approve the token to the lender for the total repayment amount, then take flashloan
    /// @dev Initiate a flash loan
    function flashBorrow(address token, uint256 amount) public {
        bytes memory data = abi.encode(Action.NORMAL);

        uint256 _allowance = IERC20(token).allowance(address(this), address(lender));
        uint256 _fee = lender.flashFee(token, amount);
        uint256 _repayment = amount + _fee;

        IERC20(token).approve(address(lender), _allowance + _repayment);
        lender.flashLoan(this, token, amount, data);
    }


/*
Now of course we will also need the onFlashLoan callback function in our borrower. 

In our example we

1. verify the sender is actually the lender
2. verify the initiator for the flash loan was actually our contract
3. return the pre-defined hash to verify a successful flash loan

We could further implement additional logic here based on the passed data field if required.

Now we can go into the lender implementation where we will execute the actual flash loan logic.
*/
    /// @dev ERC-3156 Flash loan callback
    function onFlashLoan(address initiator, address token, uint256 amount, uint256 fee, bytes calldata data) external override returns(bytes32) {
        require(msg.sender == address(lender), "FlashBorrower: Untrusted lender");
        require(initiator == address(this), "FlashBorrower: Untrusted loan initiator");
        
        // optionally check data here if wanted
        (Action action) = abi.decode(data, (Action));
        if (action == Action.NORMAL) {
            // do one thingol
        } else if (action == Action.OTHER) {
            // do another
        }
        return keccak256("ERC3156FlashBorrower.onFlashLoan");
    }
}
```
{% endtab %}

{% tab title="FlashLender.sol" %}
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "lib/yield-utils-v2/contracts/mocks/ERC20Mock.sol";
import "lib/yield-utils-v2/contracts/token/IERC20.sol";

import "src/IERC3156FlashLender.sol";
import "src/IERC3156FlashBorrower.sol";

/**
@title Flash Loan Server
@author Calnix
@dev Contract allows users to exchange a pre-specified ERC20 token for some other wrapped ERC20 tokens.
@notice Wrapped tokens will be burned, when user withdraws their deposited tokens.
*/

contract FlashLoanVault is ERC20Mock, IERC3156FlashLender {

    ///@dev The keccak256 hash of "ERC3156FlashBorrower.onFlashLoan"
    bytes32 public constant CALLBACK_SUCCESS = keccak256("ERC3156FlashBorrower.onFlashLoan");

    ///@notice Fee is a constant 0.1%
    ///@dev 1000 == 0.1%  (1 == 0.0001 %)
    uint256 public constant fee = 1000; 

    ///@dev mapping of supported addresses by Flash loan provider
    mapping(address => bool) public supportedTokens;

    ///@dev ERC20 interface specifying token contract functions
    IERC20 public immutable underlying;    

    ///@notice Creates a new wrapper token for a specified token 
    ///@dev Token will have 18 decimal places as ERC20Mock inherits from ERC20Permit
    ///@param underlying_ Address of underlying ERC20 token (e.g. DAI)
    ///@param tokenName Name of FractionalWrapper tokens 
    ///@param tokenSymbol Symbol of FractionalWrapper tokens (e.g. yvDAI)
    constructor(IERC20 underlying_, string memory tokenName, string memory tokenSymbol) ERC20Mock(tokenName, tokenSymbol) {
        underlying = underlying_;
        supportedTokens[address(underlying_)] = true;
    }


    /*/////////////////////////////////////////////////////////////*/
    /*                            FLASHLOAN                        */
    /*/////////////////////////////////////////////////////////////*/
    

    /** Note: The flashLoan function MUST include a callback to the onFlashLoan function in a IERC3156FlashBorrower contract.
     * @dev Loan `amount` tokens to `receiver`, and takes it back plus a `flashFee` after the callback.
     * @param receiver The contract receiving the tokens, needs to implement the `onFlashLoan(address user, uint256 amount, uint256 fee, bytes calldata)` interface.
     * @param token The loan currency.
     * @param amount The amount of tokens lent.
     * @param data A data parameter to be passed on to the `receiver` for any custom use.
     */
    function flashLoan(IERC3156FlashBorrower receiver, address token, uint256 amount, bytes calldata data) external returns(bool) {
        require(supportedTokens[token], "FlashLender: Unsupported currency");

        uint256 _fee = _flashFee(token, amount);
        require(IERC20(token).transfer(address(receiver), amount), "FlashLender: Transfer failed");

        require(receiver.onFlashLoan(msg.sender, token, amount, _fee, data) == CALLBACK_SUCCESS, "IERC3156: Callback failed");
        require(IERC20(token).transferFrom(address(receiver), address(this), amount + _fee), "FlashLender: Repay failed");

        return true;
    }
        

    //Note: The flashFee function MUST return the fee charged for a loan of amount token. If the token is not supported flashFee MUST revert.
    ///@dev The fee to be charged for a given loan.
    ///@param token The loan currency.
    ///@param amount The amount of tokens lent.
    ///@return The amount of `token` to be charged for the loan, on top of the returned principal.
    function flashFee(address token, uint256 amount) external view returns (uint256) {
        require(supportedTokens[token], "FlashLender: Unsupported currency");
        return _flashFee(token, amount);
    }


    ///@dev The fee to be charged for a given loan. Internal function with no checks.
    ///@param token The loan currency.
    ///@param amount The amount of tokens lent.
    ///@return The amount of `token` to be charged for the loan, on top of the returned principal.
    ///Note: division of 10000 is for rebasing fee from its integer to percentage form.
    function _flashFee(address token, uint256 amount) internal view returns (uint256) {
        return amount * fee / 10000;
    }

    ///Note: The maxFlashLoan function MUST return the maximum loan possible for token. If a token is not currently supported maxFlashLoan MUST return 0, instead of reverting.
    ///@dev The amount of currency available to be lended.
    ///@param token The loan currency.
    ///@return The amount of `token` that can be borrowed -> max of DAI deposited.
    function maxFlashLoan(address token) external view returns (uint256) {
        return supportedTokens[token] ? IERC20(token).balanceOf(address(this)) : 0;
    }
```
{% endtab %}
{% endtabs %}

