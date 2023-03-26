carrot

medium

# Non-consecutive `tokenIds` can be issued for a loan

## Summary

Loans can be issued out of order.

## Vulnerability Detail

The contract tracks 2 values for each loan object: `supplyPositionIndex` and `nbOfPositions`. These store the first `tokenId`s of the loan NFTs against which the this loan was created. The contract also assumes that the loan NFTs for a particular loan are in order and consecutive, as is evident from this comment

https://github.com/sherlock-audit/2023-02-kairos/blob/main/kairos-contracts/src/DataStructure/Storage.sol#L48-L49

From this information we can assume that the UI intends to show the corresponding loan NFTs for a particular loan by assuming that the matched loans are the `tokenId`s ranging from `supplyPositionIndex` to `supplyPositionIndex`+`nbOfPositions`-1, or in other words, consecutive. However this assumption is not true, and thus can lead to wrong display of loans in the UI.

This can be violated due to the lack of `nonreentrant` modifiers on the contract. Whenever an offer is used, it calls `safemint()` which hands over control to the receiver of the NFT.

https://github.com/sherlock-audit/2023-02-kairos/blob/main/kairos-contracts/src/BorrowLogic/BorrowHandlers.sol#L91

The re-entrancy can be performed by both the borrower or the loan NFT receiver, to sow confusion in the UI. For this case, lets assume the loan provider is re-entering the contract.

Steps to re-create:

1. Borrower (A) uses 4 offers. Contract iterates over the offers and mints NFTs for each offer. When `safeMint` is called, the loan NFT receiver (B) gets minted NFT with token id `10` and then gets control through their `onERC721Received` callback function.
2. B calls `borrow` with some other NFT and offers and mints `tokenId`s `11`,`12`,`13` for the loans. Finishes execution and hands back control.
3. A's borrow offer finishes execution and mints tokens `14`,`15`,`16`.
4. A's position is matched with tokenids `10`,`14`,`15`,`16`. Since `supplyPositionIndex` is set to `10` and `nbOfPositions` is set to `4`, the UI might show the loans as `10`,`11`,`12`,`13` instead, which would be incorrect.

Since this violates the assumption made in the linked comment, and shows the code behaves in a way different to how the developer intended, this is classified as a medium bug.

## Impact

Incorrect accounting of loan NFTs for a loan.

## Code Snippet

https://github.com/sherlock-audit/2023-02-kairos/blob/main/kairos-contracts/src/BorrowLogic/BorrowHandlers.sol#L91

## Tool used

Manual Review

## Recommendation

Use nonreentrant modifiers on functions which interact with the NFTs.
