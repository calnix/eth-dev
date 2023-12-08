# Project #5: Collateralized Vault

## Objective

* Contract that acts as a collateralized debt engine.&#x20;
* Contract allows users to deposit an asset they own (collateral), to **borrow a different asset** that the Vault owns (underlying).&#x20;
* Exchange rate determined by oracle. If value of collateral drops in underlying terms, the user will be liquidated.

## Tokens

* Collateral: WETH&#x20;
* Underlying: DAI

## Contract

1. Pull the contract code for both from Etherscan
2. Add a `mint()` function that allows you to obtain as much as you need for testing.

## Workflow

1. Deposit: User deposit WETH into Vault. Vault records WETH deposited.
2. Borrow: User _borrows_ DAI against their WETH deposit.
   * As long as the value DAI they borrow in WETH terms is less than the value of their WETH collateral
   * DAI\_value\_in\_WETH < WETH deposit
   * Vault transfers DAI to the users
   * Vault owner finances DAI to the Vault on deployment
3. Exchange rate: Chainlink Oracle \[[https://docs.chain.link/docs/ethereum-addresses](https://docs.chain.link/docs/ethereum-addresses)]
4. Withdrawal
   * To withdraw WETH, the users must repay the DAI they borrowed.
5. Liquidation
   * If ETH/DAI price changes such tt **debt\_value** _>_ **collateral\_value**, Vault will erase user records from the contract -> cancelling the user debt, and at the same time stopping that user from withdrawing their collateral.

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

## Process

#### Dependencies

* IERC20 interface): `forge install yieldprotocol/yield-utils-v2`&#x20;
* Ownable.sol: `forge install openZeppelin/openzeppelin-contracts`
* AggregatorV3 and MockV3Aggregator: `forge install smartcontractkit/chainlink`

### Pull contracts from Etherscan

* WETH9.sol: [https://etherscan.io/address/0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2#code](https://etherscan.io/address/0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2#code)&#x20;
* DAI.sol: [https://etherscan.io/address/0x6b175474e89094c44da98b954eedeac495271d0f#code](https://etherscan.io/address/0x6b175474e89094c44da98b954eedeac495271d0f#code)

### Updating contracts to v0.8.0

**WETH9.sol:**

1. Added `emit` to all events
2. Line 70: `uint(-1)` refactored to `type(uint).max`&#x20;

**DAI.sol:**

* removed inheritance of LibNote&#x20;
* line 78 & 79: removed note modifier from function reply and deny
* line 112: removed `public` from constructor.&#x20;
* line 190: `now` deprecated, changed to `block.timestamp`
* line 192: `uint(-1)` changed to `type(uint).max` (also on 131, 147)

## Pricing + Decimal scaling

<figure><img src="https://camo.githubusercontent.com/400589b8939f6f7390f1e3e08316d2f674e7709cdcd6df6f6879873f88570f34/68747470733a2f2f313733333838353834332d66696c65732e676974626f6f6b2e696f2f7e2f66696c65732f76302f622f676974626f6f6b2d782d70726f642e61707073706f742e636f6d2f6f2f73706163657325324654676f6d7a6c6d6e394e72785559304f5133634425324675706c6f61647325324674304b6b76426b477430535745425a58446f3532253246696d6167652e706e673f616c743d6d6564696126746f6b656e3d30643339613835362d376530372d346234302d396336372d663461653036653962653737" alt=""><figcaption></figcaption></figure>

## Testing

#### Action: deposit, borrow, repay, withdraw. liquidated

StateZero:(Vault has 10000 DAI)

* testVaultHasDAI
* User deposits WETH

> * cannotWithdraw -> nothing to withdraw. no need to test.

StateDeposited: (Vault has 10000 DAI, 1 WETH) | (user deposited 1 WETH. can borrow, withdraw)

* userCannotWithdrawExcess
* userCannotBeLiquidatedWithoutDebt
* userCannotBorrowInExcess
* User can withdraw freely in absence of debt (fuzzing)
* User can borrow against collateral provided

StateBorrowed: (user has borrowed half of maxDebt. actions:\[borrow,repay,withdraw])

* userCannotBorrowExcessOfMargin
* userCannotWithdrawExcessOfMargin
* userCannotbeLiquidated - price unchanged.
* User Repays (Fuzzing)

StateLiquidation (setup price to exceed)

* userLiquidated
* testOnlyOwnerCanCallLiquidate
