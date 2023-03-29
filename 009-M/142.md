csanuragjain

medium

# ACL missing on init function

## Summary
The `init` method can be called by anyone to reset system state.

## Vulnerability Detail
1. Observe the `init` function

```solidity
function init() external {
        // adding ERC165 data
        LibDiamond.DiamondStorage storage ds = LibDiamond.diamondStorage();
        ds.supportedInterfaces[type(IERC165).interfaceId] = true;
        ds.supportedInterfaces[type(IDiamondCut).interfaceId] = true;
        ds.supportedInterfaces[type(IDiamondLoupe).interfaceId] = true;
        ds.supportedInterfaces[type(IERC173).interfaceId] = true;

        // initializing protocol
        Protocol storage proto = protocolStorage();

        proto.tranche[0] = ONE.div(10).mul(4).div(365 days); // 40% APR
        proto.nbOfTranches = 1;
        proto.auction.priceFactor = ONE.mul(3);
        proto.auction.duration = 3 days;

        // initializing supply position nft collection
        SupplyPosition storage sp = supplyPositionStorage();
        sp.name = "Kairos Supply Position";
        sp.symbol = "KSP";
        ds.supportedInterfaces[type(IERC721).interfaceId] = true;
    }
```

2. Notice this function can be called by anyone and also resets all important variables to default value
3. Lets say if Admin has created a new tranche, this increases the proto.nbOfTranches to 2 using [createTranche](https://github.com/sherlock-audit/2023-02-kairos/blob/main/kairos-contracts/src/AdminFacet.sol#L47) function

```solidity
function createTranche(Ray newTranche) external onlyOwner returns (uint256 newTrancheId) {
        Protocol storage proto = protocolStorage();

        newTrancheId = proto.nbOfTranches++;
        proto.tranche[newTrancheId] = newTranche;

        emit NewTranche(newTranche, newTrancheId);
    }
```

4. Now Attacker can simply call `init` function and proto.nbOfTranches will be reset back to 1 even though it should be 2. 
5. So now if Admin adds a new tranche it will overwrite his previous added tranche since `proto.nbOfTranches++` will result in 2 instead of expected value of `3`

## Impact
Important variables like `proto.nbOfTranches` and auction parameters could be reset back to default value of 1. POC shows scenario where Attacker overwrites any tranche created by Admin

## Code Snippet
https://github.com/sherlock-audit/2023-02-kairos/blob/main/kairos-contracts/src/Initializer.sol#L24

## Tool used
Manual Review

## Recommendation
Implement ACL on `init` function so that it is only callable by owner