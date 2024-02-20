# Gas Optimizations

## Summary

| ID | Description |
| :-: | - |
| [G-01](#g-01) | Recommend removing the redundant logic |
| [G-02](#g-02) | Recommend refactoring to execute point addition logic for nonzero values |
| [G-03](#g-03) | Recommend reusing the functionality of `AccessControl` for role management |
| [G-04](#g-04) | Recommend enhancing `GameItems` contract with `burnBatch` functionality |
| [G-05](#g-05) | Recommend implementing separate reward claiming functionality within the `MergingPool` contract |
| [G-06](#g-06) | Recommend refactoring FighterFarm::_createNewFighter() to implement Inline Emission |
</br>

## **[G-01] Recommend removing the redundant logic**<a name="g-01"></a>

The [AiArenaHelper::constructor()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L41-L52) contains redundant logic, specifically at line [AiArenaHelper::L49](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L49), where attribute probabilities are set. 

```solidity
File: AiArenaHelper.sol

49: attributeProbabilities[0][attributes[i]] = probabilities[i];
```

This logic redundant the functionality found in the [AiArenaHelper::addAttributeProbabilities()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131-L139).

```solidity
File: AiArenaHelper.sol

function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
-->         attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
    }
```

*There is one instance of this issue:*

```solidity
File: AiArenaHelper.sol

    constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 
```
</br>

---

## **[G-02] Recommend refactoring to execute point addition logic for nonzero values**<a name="g-02"></a>

The point addition logic at [RankedBattle::L466-468](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L466-L468) and [RankedBattle::L485-487](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L485-L487) can be optimized by executing it conditionally, where `points > 0`, to reduce gas consumption from unnecessary addition or subtraction operations when the amount is zero.

*There is two instances of this issue:*

```solidity
File: RankedBattle.sol

    function _addResultPoints(
        uint8 battleResult, 
        uint256 tokenId, 
        uint256 eloFactor, 
        uint256 mergingPortion,
        address fighterOwner
    ) 
        private 
    {
        // SNIPPED

        if (battleResult == 0) {
            /// If the user won the match

            /// If the user has no NRNs at risk, then they can earn points
            if (stakeAtRisk == 0) {
                points = stakingFactor[tokenId] * eloFactor;
            }

            /// Divert a portion of the points to the merging pool
            uint256 mergingPoints = (points * mergingPortion) / 100;
            points -= mergingPoints;
            _mergingPoolInstance.addPoints(tokenId, mergingPoints);

            /// Do not allow users to reclaim more NRNs than they have at risk
            if (curStakeAtRisk > stakeAtRisk) {
                curStakeAtRisk = stakeAtRisk;
            }

            /// If the user has stake-at-risk for their fighter, reclaim a portion
            /// Reclaiming stake-at-risk puts the NRN back into their staking pool
            if (curStakeAtRisk > 0) {
                _stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, tokenId, fighterOwner);
                amountStaked[tokenId] += curStakeAtRisk;
            }

            /// Add points to the fighter for this round
-->         accumulatedPointsPerFighter[tokenId][roundId] += points;
-->         accumulatedPointsPerAddress[fighterOwner][roundId] += points;
-->         totalAccumulatedPoints[roundId] += points;
            if (points > 0) {
                emit PointsChanged(tokenId, points, true);
            }
        } else if (battleResult == 2) {

            // SNIPPED

        }
    } 
```

```solidity
File: RankedBattle.sol

    function _addResultPoints(
        uint8 battleResult, 
        uint256 tokenId, 
        uint256 eloFactor, 
        uint256 mergingPortion,
        address fighterOwner
    ) 
        private 
    {
        // SNIPPED

        if (battleResult == 0) {

            // SNIPPED

        } else if (battleResult == 2) {
            /// If the user lost the match

            /// Do not allow users to lose more NRNs than they have in their staking pool
            if (curStakeAtRisk > amountStaked[tokenId]) {
                curStakeAtRisk = amountStaked[tokenId];
            }
            if (accumulatedPointsPerFighter[tokenId][roundId] > 0) {
                /// If the fighter has a positive point balance for this round, deduct points 
                points = stakingFactor[tokenId] * eloFactor;
                if (points > accumulatedPointsPerFighter[tokenId][roundId]) {
                    points = accumulatedPointsPerFighter[tokenId][roundId];
                }
-->             accumulatedPointsPerFighter[tokenId][roundId] -= points;
-->             accumulatedPointsPerAddress[fighterOwner][roundId] -= points;
-->             totalAccumulatedPoints[roundId] -= points;
                if (points > 0) {
                    emit PointsChanged(tokenId, points, false);
                }
            } else {
                /// If the fighter does not have any points for this round, NRNs become at risk of being lost
                bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
                if (success) {
                    _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
                    amountStaked[tokenId] -= curStakeAtRisk;
                }
            }
        }
    } 
```
</br>

---

## **[G-03] Recommend reusing the functionality of `AccessControl` for role management**<a name="g-03"></a>

Since the [Neuron](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol) contract inherits the [AccessControl](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/access/AccessControl.sol), it can reuse the provided [AccessControl::grantRole()](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/access/AccessControl.sol#L144-L146) to assign specific roles to certain addresses instead of implementing a separate role assignment function.

This approach can reduce the contract size and deployment gas costs.

*There is one instance of this issue:*

```solidity
File: Neuron.sol

    contract Neuron is ERC20, AccessControl {

        // SNIPPED

    }
```
</br>

---

## **[G-04] Recommend enhancing `GameItems` contract with `burnBatch` functionality**<a name="g-04"></a>

As the [GameItems](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol) contract introduce the ability to burn items, it can be improved by implementing the external `GameItems::burnBatch()` to support safer gas consumption during batch burning operations for multiple items.

The contract that can be used as a reference:
* [ERC1155Burnable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC1155/extensions/ERC1155Burnable.sol)

*There is one instance of this issue:*

```solidity
File: GameItems.sol

    contract GameItems is ERC1155 {

        // SNIPPED

    }
```
</br>

---

## **[G-05] Recommend implementing separate reward claiming functionality within the `MergingPool` contract**<a name="g-05"></a>

As the [MergingPool](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol) contract introduces the ability to batch claim rewards for multiple rounds, it can be improved by implementing the external `MergingPool::claimReward()` to support claiming rewards for specific rounds. 

Consider the scenario where a player has never won in many previous rounds. To claim their rewards, they would need to iterate through all rounds and winners arrays, resulting in significant gas consumption.

This improvement would provide safer gas consumption and prevent denial of service scenarios, particularly in cases where a winner has not won in many previous rounds.

*There is one instance of this issue:*

```solidity
File: MergingPool.sol

    function claimRewards(
        string[] calldata modelURIs, 
        string[] calldata modelTypes,
        uint256[2][] calldata customAttributes
    ) 
        external 
    {
        uint256 winnersLength;
        uint32 claimIndex = 0;
        uint32 lowerBound = numRoundsClaimed[msg.sender];
-->     for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            numRoundsClaimed[msg.sender] += 1;
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
        if (claimIndex > 0) {
            emit Claimed(msg.sender, claimIndex);
        }
    }
```
</br>

---

## **[G-06] Recommend refactoring FighterFarm::_createNewFighter() to implement Inline Emission**<a name="g-06"></a>

As the [FighterFarm::fighterCreatedEmitter()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L53-L62) is solely used to emit an event when a fighter is created within [FighterFarm::_createNewFighter()#L530](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L530), the [FighterFarm::_createNewFighter()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L484-L531) function can be optimized by refactoring to inline the emission.

*There is one instance of this issue:*

```solidity
File: FighterFarm.sol

    function _createNewFighter(
        address to, 
        uint256 dna, 
        string memory modelHash,
        string memory modelType, 
        uint8 fighterType,
        uint8 iconsType,
        uint256[2] memory customAttributes
    ) 
        private 
    {  
        require(balanceOf(to) < MAX_FIGHTERS_ALLOWED);
        uint256 element; 
        uint256 weight;
        uint256 newDna;
        if (customAttributes[0] == 100) {
            (element, weight, newDna) = _createFighterBase(dna, fighterType);
        }
        else {
            element = customAttributes[0];
            weight = customAttributes[1];
            newDna = dna;
        }
        uint256 newId = fighters.length;

        bool dendroidBool = fighterType == 1;
        FighterOps.FighterPhysicalAttributes memory attrs = _aiArenaHelperInstance.createPhysicalAttributes(
            newDna,
            generation[fighterType],
            iconsType,
            dendroidBool
        );
        fighters.push(
            FighterOps.Fighter(
                weight,
                element,
                attrs,
                newId,
                modelHash,
                modelType,
                generation[fighterType],
                iconsType,
                dendroidBool
            )
        );
        _safeMint(to, newId);
-->     FighterOps.fighterCreatedEmitter(newId, weight, element, generation[fighterType]);
    }
```
</br>

---