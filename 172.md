duc

medium

# Borrower can reduce the interest of shortly loans

## Summary
Borrower can reduce the interest of quick loans by using multiple own offers.
## Vulnerability Detail
* If the borrower repay the loan after a short time elapsed, `interests` of loan will be `loan.payment.minInterestsToRepay`.
* When lenders claim the loan in case `interests == loan.payment.minInterestsToRepay`, the interest will be equally distributed for all lenders.
```solidity=
function sendInterests(Loan storage loan, Provision storage provision) internal returns (uint256 sent) {
    uint256 interests = loan.payment.paid - loan.lent;
    if (interests == loan.payment.minInterestsToRepay) {
        // this is the case if the loan is repaid shortly after issuance
        // each lender gets its minimal interest, as an anti ddos measure to spam offer
        sent = provision.amount + (interests / loan.nbOfPositions);
        //[audit-med] borrower can use multiple offers to reduce interest
    } else {
        /* provision.amount / lent = share of the interests belonging to the lender. The parenthesis make the
        calculus in the order that maximizes precison */
        sent = provision.amount + (interests * (provision.amount)) / loan.lent;
    }
    loan.assetLent.checkedTransfer(msg.sender, sent);
}
```
* Therefore, the borrower can reduce this interest to be very small, by using multiple own offers with 1 wei of amount. Because `loan.nbOfPositions` will be very large.
* Scenerio:
    1. Alice has an offer for a collateral NFT with `loanToValue` = 1e18 
    2. Bob owns the collateral NFT, and he creates 9 offers with same `loanToValue`
    3. Bob uses his NFT to loan quickly (or flashloan) assets from Alice's offer and his 9 offers. He passes amount of Alice's offer = 1e18 - 9, and each amount of Bob's offer = 1
    4. Bob repays right after the loan, the interest of loan will be `loan.payment.minInterestsToRepay`. Then he can claim with 9 positions from 9 offers, and get 9/10 of the interest. Alice's gets only 1/10 of the interest
## Impact
The borrowers can reduce the interest to be paid for their quick loans.

## Code Snippet
https://github.com/sherlock-audit/2023-02-kairos/blob/main/kairos-contracts/src/ClaimFacet.sol#L99
## Tool used
Manual review

## Recommendation
Should distribute the minimum interest based on the amount of each offer (povision.amount)