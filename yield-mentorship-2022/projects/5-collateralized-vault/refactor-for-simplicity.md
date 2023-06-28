# Refactor for Simplicity

remove getMaxDebt -> use \_isCollateralized -> getCollateralRequired is replaced by \_isCollateralized

modify borrow() -> calls \_isCollateralized

modify withdraw() ->

modify liquidation -> use \_isCollateralized
