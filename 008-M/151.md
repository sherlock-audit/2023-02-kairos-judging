tsvetanovv

medium

# ECDSA.recover Signature Malleability

## Summary

In [BorrowCheckers.sol](https://github.com/sherlock-audit/2023-02-kairos/blob/main/kairos-contracts/src/BorrowLogic/BorrowCheckers.sol#L29) we have `checkOfferArg()` function. This function have Signature Malleability.

```solidity
signer = ECDSA.recover(offerDigest(arg.offer), arg.signature);
```

## Vulnerability Detail

OpenZeppelin has a vulnerability in versions lower than 4.7.3, which can be exploited by an attacker. The project uses a vulnerable version.
The protocol currently uses `^4.6.0`.

The functions `ECDSA.recover` and `ECDSA.tryRecover` are vulnerable to a kind of signature malleability due to accepting EIP-2098 compact signatures in addition to the traditional 65 byte signature format. This is only an issue for the functions that take a single `bytes` argument, and not the functions that take `r, v, s` or `r, vs` as separate arguments.

The potentially affected contracts are those that implement signature reuse or replay protection by marking the signature itself as used rather than the signed message or a nonce included in it. A user may take a signature that has already been submitted, submit it again in a different form, and bypass this protection.

[Reference](https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-4h98-2769-gh6h)

## Impact

See Vulnerability Detail

## Code Snippet

https://github.com/sherlock-audit/2023-02-kairos/blob/main/kairos-contracts/src/BorrowLogic/BorrowCheckers.sol#L29

```solidity
signer = ECDSA.recover(offerDigest(arg.offer), arg.signature);
```

## Tool used

Manual Review

## Recommendation

Upgrade `@openzeppelin/contracts` to the latest version or a version that is newer or at least 4.7.3.