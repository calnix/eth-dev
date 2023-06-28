# Full code for lender

## Lender that is compliant to ERC 3156 and 4626

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "lib/yield-utils-v2/contracts/mocks/ERC20Mock.sol";
//import "lib/openzeppelin-contracts/contracts/access/Ownable.sol";

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


    /*/////////////////////////////////////////////////////////////*/
    /*                       FRACTIONAL WRAPPER                    */ 
    /*/////////////////////////////////////////////////////////////*/


    ///@dev Exchange rate at inception: 1 underlying (DAI) == 1 share (yvDAI) | Ex-rate: 1 DAI/yvDAI = 0.5 -> 1 DAI gets you 1/2 yvDAI
    //uint exRate = 1e27;

    /// @notice Emit event when ERC20 tokens are deposited into Fractionalized Wrapper
    event Deposit(address indexed caller, address indexed owner, uint256 assets, uint256 shares);
    
    /// @notice Emit event when ERC20 tokens are withdrawn from Fractionalized Wrapper
    event Withdraw(address indexed caller, address indexed receiver, address indexed owner, uint256 assets, uint256 shares);


    /// @notice User to deposit underlying tokens for FractionalWrapper tokens
    /// @dev Mints shares Vault shares to receiver by depositing exactly amount of underlying tokens
    /// @param assets The amount of underlying tokens to deposit
    /// @param receiver Address of receiver of Fractional Wrapper shares
    function deposit(uint256 assets, address receiver) external returns(uint256 shares) {       
        receiver = msg.sender;
        shares = convertToShares(assets);

        //transfer DAI from user
        bool success = underlying.transferFrom(receiver, address(this), assets);
        require(success, "Deposit failed!");   

        //mint yvDAI to user
        bool sent = _mint(receiver, shares);
        require(sent, "Mint failed!"); 

        emit Deposit(msg.sender, receiver, assets, shares);  
    }


    /// @dev Burns shares from owner and sends exactly assets of underlying tokens to receiver; based on the exchange rate.
    /// @param assets The amount of underlying tokens to withdraw
    /// @param receiver Address of receiver of underlying tokens - DAI
    /// @param owner Address of owner of Fractional Wrapper shares - yvDAI
    function withdraw(uint256 assets, address receiver, address owner) external returns(uint256 shares) {
        shares = convertToShares(assets);
        
        // MUST support a withdraw flow where the shares are burned from owner directly where owner is msg.sender,
        // OR msg.sender has ERC-20 approval over the shares of owner
        if(msg.sender != owner){
            uint allowedShares = _allowance[owner][receiver] ;
            require(allowedShares >= shares, "Allowance exceeded!");
            _allowance[owner][receiver] = allowedShares - shares;
        }

        //burn wrapped tokens(shares) -> yvDAI 
        burn(owner, shares);
        
        //transfer assets
        bool success = underlying.transfer(receiver, assets);
        require(success, "Transfer failed!"); 
        emit Withdraw(msg.sender, receiver, owner, assets, shares);
    }

    /// @dev Burns shares from owner and sends assets of underlying tokens to receiver; based on the exchange rate.
    /// @param shares The amount of wrapped tokens to redeem for underlying tokens (assets)
    /// @param receiver Address of receiver of underlying tokens - DAI
    /// @param owner Address of owner of Fractional Wrapper shares - yvDAI
    function redeem(uint256 shares, address receiver, address owner) external returns(uint256 assets) {
        assets = convertToAssets(shares);
        
        // MUST support a redeem flow where the shares are burned from owner directly where owner is msg.sender,
        // OR msg.sender has ERC-20 approval over the shares of owner
        if(msg.sender != owner){
            uint allowedShares = _allowance[owner][receiver] ;
            require(allowedShares >= shares, "Allowance exceeded!");
            _allowance[owner][receiver] = allowedShares - shares;
        }

        //burn wrapped tokens(shares) -> yvDAI 
        burn(owner, shares);
        
        //transfer assets
        bool success = underlying.transfer(receiver, assets);
        require(success, "Transfer failed!"); 
        emit Withdraw(msg.sender, receiver, owner, assets, shares);
    }



    /// @notice Returns the unit 'exchange rate'; assuming 1 unit of underlying was deposited, how much shares would be received 
    /// @dev wrapperMinted = (underlyingDeposited,(1) * wrapperSupply) / underlyingInWrapper 
    /// Note: Exchange rate is floating, it's dynamic based on capital in/out-flows
    /// Note: _totalSupply returns a value extended over decimal precision, in this case 18 dp. hence the scaling before divsion.
    function getExchangeRate() internal view returns(uint256) {
        uint256 sharesMinted;
        return _totalSupply == 0 ? sharesMinted = 1e18 : sharesMinted = (_totalSupply * 1e18) / underlying.balanceOf(address(this));
    //    if(_totalSupply == 0) {
    //        return sharesMinted = 1;
    //    }
    //    return sharesMinted = (_totalSupply) / underlying.balanceOf(address(this));
    }

    /// @notice Calculate how much yvDAI user should get based on flaoting exchange rate
    /// @dev getExchangeRate() returns shares minted per unit of underlying asset deposited; at present moment.
    /// @param assets Amount of underlying tokens (assets) to be converted to wrapped tokens (shares)
    function convertToShares(uint256 assets) internal view returns(uint256 shares){
        return assets * getExchangeRate() / 1e18;
    }

    /// @notice Calculate how much DAI user should get based on floating exchange rate
    /// @dev getExchangeRate() returns shares minted per unit of underlying asset deposited; at present moment.
    /// @param shares Amount of wrapped tokens (shares) to be converted to underlying tokens (assets) 
    function convertToAssets(uint256 shares) internal view returns(uint256 assets){
        return shares * 1e18 / getExchangeRate();
    }

    /// @notice Allows an on-chain or off-chain user to simulate the effects of their deposit at the current block, given current on-chain conditions
    /// @dev Any unfavorable discrepancy between convertToShares and previewDeposit SHOULD be considered slippage
    /// @param assets Amount of assets to deposit
    function previewDeposit(uint256 assets) external view returns(uint256 shares) {
        return convertToShares(assets);
    }


    /// @notice Allows an on-chain or off-chain user to simulate the effects of their withdrawal at the current block, given current on-chain conditions.
    /// @dev Any unfavorable discrepancy between convertToShares and previewDeposit SHOULD be considered slippage
    /// @param assets Amount of assets to withdraw
    function previewWithdraw(uint256 assets) external view returns(uint256 shares) {
        shares = convertToShares(assets);
    }

   
    ///Note: newly added

    ///@notice Function returns contract address of underlying token utilized by the vault (e.g. DAI)
    ///@dev MUST be an ERC-20 token contract 
    function asset() external view returns(address assetTokenAddress){
        assetTokenAddress = address(underlying);
    }

    ///@notice Returns total amount of the underlying asset (e.g. DAI) that is “managed” by Vault.
    ///@dev SHOULD include any compounding that occurs from yield, account for any fees that are charged against assets in the Vault
    function totalAssets() external view returns(uint256 totalManagedAssets) {
        totalManagedAssets = underlying.balanceOf(address(this));
    }


    ///@notice Maximum amount of the underlying asset that can be deposited into the Vault by the receiver, through a deposit call.
    ///@dev Return the maximum amount of assets that would be allowed for deposit by receiver, and not cause revert
    ///Note: In this Vault implementation there are no restrictions in minting supply, therefore, no deposit restrictions. 
    ///Note: Consequently, maxAssets = type(uint256).max for all users => therefore we drop the 'receiver' param as specified in EIP4626
    ///Note: To allow for this to be overwritten as per EIP4626, function is set to virtual
    function maxDeposit() external view virtual returns (uint256 maxAssets) {
        maxAssets = type(uint256).max;
    }
    
    ///@notice Maximum amount of shares that can be minted from the Vault for the receiver, through a mint call.
    ///@dev MUST return the maximum amount of shares mint would allow to be deposited to receiver+
    ///Note: In this Vault implementation there are no restrictions in minting supply
    ///Note: Consequently, maxShares = type(uint256).max for all users => therefore we drop the 'receiver' param as specified in EIP4626
    ///Note: To allow for this to be overwritten as per EIP4626, function is set to virtual
    function maxMint() external view virtual returns(uint maxShares) {
        maxShares = type(uint256).max;
    }

    ///@notice Allows a user to simulate the effects of their mint at the current block, given current on-chain conditions.
    ///@dev Return as close to and no fewer than the exact amount of assets that would be deposited in a mint call in the same transaction
    ///@param shares Amount of shares to be minted
    function previewMint(uint256 shares) external view returns (uint256 assets) {
        assets = convertToAssets(shares);
    }


    ///@notice Mints Vault shares to receiver based on the deposited amount of underlying tokens
    ///@param shares Amount of shares to be minted
    ///@param receiver Address of receiver
    function mint(uint256 shares, address receiver) external returns (uint256 assets) {
        assets = convertToAssets(shares); 
        bool sent = underlying.transferFrom(msg.sender, address(this), assets);
        require(sent, "Transfer failed!"); 

        bool success = _mint(receiver, shares);
        require(success, "Mint failed!"); 
        emit Deposit(msg.sender, receiver, assets, shares);
    }

    ///@notice Maximum amount of the underlying asset that can be withdrawn from the owner balance in the Vault, through a withdraw call.
    ///@param owner Address of owner of assets
    function maxWithdraw(address owner) external view returns(uint256 maxAssets) {
        maxAssets = convertToAssets(_balanceOf[owner]);
    }


    ///@notice Maximum amount of Vault shares that can be redeemed from the owner balance in the Vault, through a redeem call.
    ///@param owner Address of owner of assets
    function maxRedeem(address owner) external view returns(uint256 maxShares) {
        maxShares = _balanceOf[owner];
    }

    ///@notice Allows an on-chain or off-chain user to simulate the effects of their redeemption at the current block, given current on-chain conditions.
    ///@param shares Amount of shares to be redeemed
    function previewRedeem(uint256 shares) external view returns(uint256 assets) {
        assets = convertToAssets(shares);
    }
}
```
