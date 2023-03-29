bin2chen

medium

# IERC20.transfer/transferFrom does not support all ERC20 token

## Summary
Some ERC20 such as USDT Missing return boolean , cannot be borrowed
## Vulnerability Detail
Token like [USDT](https://etherscan.io/address/0xdac17f958d2ee523a2206206994597c13d831ec7#contracts) known for using non-standard ERC20. ([Missing return boolean on transfer or transferFrom](https://forum.openzeppelin.com/t/can-not-call-the-function-approve-of-the-usdt-contract/2130/4)).

Contract function` checkedTransferFrom/checkedTransfer` will always revert when try to transfer this kind of tokens.

`BorrowFacet.borrow ()` and other places have called this and will fail although the balance is sufficient


```solidity
    function checkedTransferFrom(IERC20 currency, address from, address to, uint256 amount) internal {
        if (!currency.transferFrom(from, to, amount)) { //<-----USDT will always revert
            revert ERC20TransferFailed(currency, from, to);
        }
    }

    function checkedTransfer(IERC20 currency, address to, uint256 amount) internal {
        if (!currency.transfer(to, amount)) {  //<-----USDT will always revert
            revert ERC20TransferFailed(currency, address(this), to);
        }
    }
```

## Impact
Some ERC20 such as USDT cannot be borrowed

## Code Snippet
https://github.com/sherlock-audit/2023-02-kairos/blob/main/kairos-contracts/src/utils/Erc20CheckedTransfer.sol#L8-L20
## Tool used

Manual Review

## Recommendation
Use SafeERC20.safeTransfer() instead of IERC20 safeTransfer. This accepts ERC20 token with no boolean return like USDT.