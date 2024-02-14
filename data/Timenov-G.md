# AI Arena QA report by Timenov

## Summary

| |Issue|Instances|
 |-|:-|:-:|
| G-01 | Array's cached length not used optimally. | 8 |
| G-02 | Smaller uint can be used. | 1 |
| G-03 | State variable can be removed. | 1 |
| G-04 | State variable can be updated after loop. | 2 |
| G-05 | Unnecessary validation. | 1 |
| G-06 | Operation can be skipped. | 1 |

### [G-01] Array's cached length not used optimally.
The protocol has done a good job on caching the length of an array before a loop. However there are places that this can be even more gas efficient.

```diff
    function addAttributeDivisor(uint8[] memory attributeDivisors) external {
        require(msg.sender == _ownerAddress);
+       uint256 attributesLength = attributes.length;
-       require(attributeDivisors.length == attributes.length);
+       require(attributeDivisors.length == attributesLength);

-       uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeToDnaDivisor[attributes[i]] = attributeDivisors[i];
        }
    }
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L70-L72

```diff
+       uint256 attributesLength = attributes.length;
+       uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributesLength);
-       uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributes.length);

-       uint256 attributesLength = attributes.length;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L96-L98

```diff
+       uint256 mintPassIdsToBurnLength = mintpassIdsToBurn.length;
+       require(
+           mintPassIdsToBurnLength == mintPassDnas.length && 
+           mintPassIdsToBurnLength == fighterTypes.length && 
+           mintPassIdsToBurnLength == modelHashes.length &&
+           mintPassIdsToBurnLength == modelTypes.length
+       );
+       for (uint16 i = 0; i < mintPassIdsToBurnLength; i++) {
        
-       require(
-           mintpassIdsToBurn.length == mintPassDnas.length && 
-           mintPassDnas.length == fighterTypes.length && 
-           fighterTypes.length == modelHashes.length &&
-           modelHashes.length == modelTypes.length
-       );
-       for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L243-L249

```diff
+       uint256 winnersLength = winners.length;
+       require(winnersLength == winnersPerPeriod, "Incorrect number of winners");
-       require(winners.length == winnersPerPeriod, "Incorrect number of winners");
        require(!isSelectionComplete[roundId], "Winners are already selected");
-       uint256 winnersLength = winners.length;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L120-L122

### [G-02] Smaller uint can be used.
At the beggining `winnersPerPeriod = 2`. Later this can change, but still I think `uint8` will be sufficient instead of `uint256`.

```diff
+       for (uint8 i = 0; i < winnersLength; i++) {
-       for (uint256 i = 0; i < winnersLength; i++) {
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L124

### [G-03] State variable can be removed.
In `RankedBattle.sol` there is a state variable `_stakeAtRiskAddress` of type `address`. Its purpose is to hold the address of the `StakeAtRisk` contract. However there is another variable `_stakeAtRiskInstance` of type `StakeAtRisk`. They both are doing the same thing. The first one is used only once. So you can just remove it and when you need the address of the contract just cast it:

```diff
    /// The StakeAtRisk contract address.
-   address _stakeAtRiskAddress;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L69

```diff
    function setStakeAtRiskAddress(address stakeAtRiskAddress) external {
        require(msg.sender == _ownerAddress);
-       _stakeAtRiskAddress = stakeAtRiskAddress;
-       _stakeAtRiskInstance = StakeAtRisk(_stakeAtRiskAddress);
+       _stakeAtRiskInstance = StakeAtRisk(stakeAtRiskAddress);
    }
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L192-L196

```diff
+        bool success = _neuronInstance.transfer(address(_stakeAtRiskInstance), curStakeAtRisk);
-        bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L493

### [G-04] State variable can updated after loop.
In 2 functions, `numRoundsClaimed[msg.sender]` is increment by 1 every loop. This can be optimized so that the state variable is update after the loop.

```diff
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
-           numRoundsClaimed[msg.sender] += 1;
            winnersLength = winnerAddresses[currentRound].length;
            for (uint32 j = 0; j < winnersLength; j++) {
                if (msg.sender == winnerAddresses[currentRound][j]) {
                    _fighterFarmInstance.mintFromMergingPool(
                        msg.sender,
                        modelURIs[claimIndex],
                        modelTypes[claimIndex],
                        customAttributes[claimIndex]
                    );
                    claimIndex += 1;
                }
            }
        }
+       numRoundsClaimed[msg.sender] = roundId - 1;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L149-L163

```diff
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            nrnDistribution = getNrnDistribution(currentRound);
            claimableNRN += (
                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
            ) / totalAccumulatedPoints[currentRound];
-           numRoundsClaimed[msg.sender] += 1;
        }
+       numRoundsClaimed[msg.sender] = roundId - 1;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L299-L305

### [G-05] Unnecessary validation.
In `GameItems::mint` there is a validation:

```solidity
    require(
        allGameItemAttributes[tokenId].finiteSupply == false || 
        (
            allGameItemAttributes[tokenId].finiteSupply == true && 
            quantity <= allGameItemAttributes[tokenId].itemsRemaining
        )
    );
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L151-L157

`allGameItemAttributes[tokenId].finiteSupply` is first checked for `false` and then for `true`. The second check(for `true`) can be removed since if the value is not `false`, it will be `true`. The `require` should look like this:

```solidity
    require(
        allGameItemAttributes[tokenId].finiteSupply == false ||
        quantity <= allGameItemAttributes[tokenId].itemsRemaining
    );
```

### [G-06] Operation can be skipped.
In `AiArenaHelper::dnaToIndex` we get the `attributeProbabilityIndex`.

```solidity
    function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute) 
        public 
        view 
        returns (uint256 attributeProbabilityIndex) 
    {
        uint8[] memory attrProbabilities = getAttributeProbabilities(generation, attribute);
        
        uint256 cumProb = 0;
        uint256 attrProbabilitiesLength = attrProbabilities.length;
        for (uint8 i = 0; i < attrProbabilitiesLength; i++) {
            cumProb += attrProbabilities[i];
            if (cumProb >= rarityRank) {
                attributeProbabilityIndex = i + 1;
                break;
            }
        }
        return attributeProbabilityIndex;
    }
```

There are a few things that can be done. No need for return value and the `break` statement can be removed.

```diff
    function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute) 
        public 
        view 
-       returns (uint256 attributeProbabilityIndex) 
+       returns (uint256) 
    {
        uint8[] memory attrProbabilities = getAttributeProbabilities(generation, attribute);
        
        uint256 cumProb = 0;
        uint256 attrProbabilitiesLength = attrProbabilities.length;
        for (uint8 i = 0; i < attrProbabilitiesLength; i++) {
            cumProb += attrProbabilities[i];
            if (cumProb >= rarityRank) {
+               return i + 1;
-               attributeProbabilityIndex = i + 1;
-               break;
            }
        }
-       return attributeProbabilityIndex;
+       return 0;
    }
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L169-L186