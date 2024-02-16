[QA-1] - ClaimNRN will revert when everybody send their 100% points to the Merging Pool

```solidity
function claimNRN() external {
        require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
        uint256 claimableNRN = 0;
        uint256 nrnDistribution;
        uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            nrnDistribution = getNrnDistribution(currentRound);
            claimableNRN += (
                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
            ) / totalAccumulatedPoints[currentRound]; 
            numRoundsClaimed[msg.sender] += 1;
        }
        if (claimableNRN > 0) {
            amountClaimed[msg.sender] += claimableNRN;
            _neuronInstance.mint(msg.sender, claimableNRN);
            emit Claimed(msg.sender, claimableNRN);
        }
    }
```

The above function will revert when everyone decide to send their 100% points to the MergingPool since `mergingPortion`  matters in `_addResultPoints`

```solidity
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
``` 

If user won a match and his mergingPortion is 100%, all of his points are sent to mergingPool.And the line `points -= mergingPoints`, points will be 0 in that case and no points are added to the rankedBattle accounting here:

```solidity
accumulatedPointsPerFighter[tokenId][roundId] += points;
            accumulatedPointsPerAddress[fighterOwner][roundId] += points;
            totalAccumulatedPoints[roundId] += points;
```
and `totalAccumulatedPoints[roundId] += points` will be 0 in that case and so in `claimNRN()` function will revert since there will be a divison by 0

```solidity
claimableNRN +=(accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution) / totalAccumulatedPoints[currentRound]
```


[QA-2] - Check-Effect-Interaction is not followed properly in `RankedBattle.sol::stakeNRN()` function properly

```solidity
 function unstakeNRN(uint256 amount, uint256 tokenId) external {
        require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");
        if (amount > amountStaked[tokenId]) {
            amount = amountStaked[tokenId];
        }
        amountStaked[tokenId] -= amount;
        globalStakedAmount -= amount;
        stakingFactor[tokenId] = _getStakingFactor(
            tokenId, 
            _stakeAtRiskInstance.getStakeAtRisk(tokenId)
        );
        _calculatedStakingFactor[tokenId][roundId] = true;
        hasUnstaked[tokenId][roundId] = true;
        bool success = _neuronInstance.transfer(msg.sender, amount);
        if (!success) {
            if (amountStaked[tokenId] == 0) {
                _fighterFarmInstance.updateFighterStaking(tokenId, false);
            }
            emit Unstaked(msg.sender, amount);
        }
    }
```

[QA-3] - Check-Effect-Interaction is not followed properly in `StakeAtRisk.sol::reclaimNRN()` function properly which could lead to re-entrancy if the game server is hacked or compromised.

In `RankedBattle.sol::updateBattleRecord()` function is calling ``_addResultPoints()` internally which is further calling `reclaimNRN()` in Case 2) Win + stake-at-risk = Reclaim some of the stake that is at risk.

```solidity
 if (curStakeAtRisk > 0) {
                _stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, tokenId, fighterOwner); 
                amountStaked[tokenId] += curStakeAtRisk;
            }
```

Now the reclaimNRN is not actually following CEI if you see

```solidity
 function reclaimNRN(uint256 nrnToReclaim, uint256 fighterId, address fighterOwner) external { 
        require(msg.sender == _rankedBattleAddress, "Call must be from RankedBattle contract");
        require(
            stakeAtRisk[roundId][fighterId] >= nrnToReclaim, 
            "Fighter does not have enough stake at risk"
        );

     @>   bool success = _neuronInstance.transfer(_rankedBattleAddress, nrnToReclaim);
        if (success) {
            stakeAtRisk[roundId][fighterId] -= nrnToReclaim;
            totalStakeAtRisk[roundId] -= nrnToReclaim;
            amountLost[fighterOwner] -= nrnToReclaim;
            emit ReclaimedStake(fighterId, nrnToReclaim);
        }
    }
```

So there could be a chance of unfair advantage here.

[QA-4] - `GameItems.sol::remainingSupply()` function will return value of `itemsRemaining` everytime it is called for infinite supply gameItem type.

The function `remainingSupply()` returns the remaining supply of a game item with the specified tokenId which is taken from the i/p of `createGameItem()` while creating the gameItem.

```solidity
function remainingSupply(uint256 tokenId) public view returns (uint256) {
        return allGameItemAttributes[tokenId].itemsRemaining;
    }
```
It will return the same value every time in case of an infinite supply type gameItem since `itemsRemaining` is only updated in case of a finite supply and never in case of an infinite supply.

It is updated in `mint()` function but with a condition of a finite supply type gameItem:

```solidity
 if (allGameItemAttributes[tokenId].finiteSupply) {
                allGameItemAttributes[tokenId].itemsRemaining -= quantity;
            }
```

[NC-1] - `_addResultPoints()` has redundant code block.

In Case 3) Lose + positive point balance = Deduct from the point balance
                /// If the fighter has a positive point balance for this round, deduct points

There is an if condition here :
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L476C13-L478C14
```solidity
if (curStakeAtRisk > amountStaked[tokenId]) {
                curStakeAtRisk = amountStaked[tokenId];
            }
```

There is no need to check this since the user is only losing points and not staked NRNs in the case 3 which is already taken care of here :
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L482-L484

```solidity
if (points > accumulatedPointsPerFighter[tokenId][roundId]) {
                    points = accumulatedPointsPerFighter[tokenId][roundId];
                }
```


