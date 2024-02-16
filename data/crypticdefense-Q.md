## Title
Missing address validation when constructing contracts

## Impact
Contract constructor input parameters should be validated in case an incorrect address is entered. This may lead to redeployment of the contract which is expensive.

## Proof of Concept
The following line of code is taken from `FighterFarm.sol`. However, this problem is consistent in `GameItems.sol`, `MergingPool.sol`, `Neuron.sol`, `RankedBattle.sol`, `StakeAtRisk.sol`, and `VoltageManager.sol`.

```javascript
 constructor(address ownerAddress, address delegatedAddress, address treasuryAddress_)
        ERC721("AI Arena Fighter", "FTR")
    {
        _ownerAddress = ownerAddress;
        _delegatedAddress = delegatedAddress;
        treasuryAddress = treasuryAddress_;
        numElements[0] = 3;
    } 
```

## Tools Used
Manual Review

## Recommendation
Verify the addresses are correct, such as adding a zero address check.