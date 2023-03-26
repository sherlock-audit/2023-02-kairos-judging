deadrxsezzz

high

# If auction price goes to 0, NFT might become unclaimable/ stuck forever

## Summary
There are certain ERC20 tokens which revert on zero value transfers (e.g. LEND). If an NFT's price drops down to 0, nobody will be able to claim it as the transaction will always revert.

## Vulnerability Detail
The time of  `loan.auction.duration` passes. The NFT's price is 0. Alice tries to purchase/ claim it, however the ERC20 token on which the auctions is going reverts on 0 value transfers. The NFT becomes stuck forever and no one can take the rights of it.

Consider the following scenarios: 
1.
 > The ERC20 used in the auction is pausable 
 > Throughout the auction, the token gets paused. Now, the auction is still going, the price is dropping and no one is able to claim it.
 > The ERC20 doesn't get unpaused up until the auction ends.
 > Since no one was able to purchase the NFT during the auction, its price now is 0. Since the token reverts on zero value transfers, the NFT is stuck forever.

2.
 > Alice is looking at NFT auction which is coming near its end.
 > Alice is looking to purchase the NFT for as little as possible and starts monitoring the mempool, so in case someone tries to buy it, she can front-run the transaction and get the NFT herself. At this point Alice getting the NFT should be guaranteed
 > However, there aren't many other active users/ they aren't paying attention to said NFT. Little time goes by, auction ends and price is set to 0.
 > Alice is happy she can claim the NFT for free. However, the token reverts on 0 value transfers and now no one can claim it and the NFT is lost forever.

## Impact
NFT might be lost forever

## Code Snippet
https://github.com/sherlock-audit/2023-02-kairos/blob/main/kairos-contracts/src/AuctionFacet.sol#L43-#L45

## Tool used

Manual Review

## Recommendation
Address ERC20 tokens which revert on 0 value transfers. Auctions which are run with such tokens should have a minimal price of 1 wei ( instead of 0)