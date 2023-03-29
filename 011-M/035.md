carrot

high

# Missing nonce in loan signatures

## Summary
Protocol does not use a nonce in the loan signatures signed by lending parties. This means same signature can be used multiple times as long as the time is within the deadline. There is also no mechanism to cancel a loan offer.
## Vulnerability Detail
The loan signatures are built using the `Signature.sol` facet, where it uses a typed hash using the chain id and parameters of the loan. It however does not use or track any nonce.
https://github.com/sherlock-audit/2023-02-kairos/blob/main/kairos-contracts/src/Signature.sol#L41-L54
This makes it so that the same loan offers can be used multiple times. If the lender has given approval to the loan token, all tokens for which the approval was given for can be used in a loan.

This can result in multiple adverse scenarios:
1. Lender signs loan for 1000USDC and approves contract for same. Borrower borrows against the loan, pays it back, and then uses the same offer signature again before expiry. This might not have been the intent of the lender.
2. Lender signs loan for 1000 USDC, and increases approval for the same, but forgets they had previous approvals already, and current approval stands at 2000 USDC. Borrowers can use the same signature twice to withdraw 2000USDC even though lender only intended a loan of 1000USDC.
3. Lender signs an offer with loan-to-value of 3000USDC. Due to changing market conditions, they revise their offer to 2000USDC. Borrower uses the older offer to withdraw at better rates. Since there is no nonce, nothing can be updated on-chain to invalidate the last offer.
4. Lender creates 2 offers, first is a 1000USDC offer for NFT A, second a 2000USDC offer for NFT B and gives approval for 3000USDC. Users can match the NFT A offer thrice, and thus not let the lender distribute his loans over multiple collections and instead is forced to loan out on the same collection multiple times.
## Impact
Re-use of signatures
## Code Snippet
https://github.com/sherlock-audit/2023-02-kairos/blob/main/kairos-contracts/src/Signature.sol#L41-L54
## Tool used

Manual Review

## Recommendation
Add a nonce mechanism to track nonce for each lender which is also used in generating the signature. This will prevent the re-use of signatures and allow users to delete old signatures by increasing the account nonce.