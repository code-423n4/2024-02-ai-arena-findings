
The winner's result in a ranked battle will be like the following according to the documentation. 
- `$NRN staked > 0` **(Condition)**
    - Win **(Condition)**
      - `stake at risk > 0` **(Condition)**
        - Reclaim stake at risk for this round
      - `stake at risk == 0` **(Condition)**
        - Increase points

However, the implementation for effective consumption of gas has not been properly considered. 
Calculation points and calling external contract like `StakeAtRisk` will be done in both cases of winner and also update unnecessary storage variables even if points is 0 or stake at risk is 0.  

```solidity
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
            accumulatedPointsPerFighter[tokenId][roundId] += points; 
            accumulatedPointsPerAddress[fighterOwner][roundId] += points;
            totalAccumulatedPoints[roundId] += points;
            if (points > 0) {
                emit PointsChanged(tokenId, points, true);
            }
```

it can be updated like the below for effective gas consumption and QA.
```solidity
             /// If the user has no NRNs at risk, then they can earn points
            if (stakeAtRisk == 0) {
                points = stakingFactor[tokenId] * eloFactor;
                /// Divert a portion of the points to the merging pool
                uint256 mergingPoints = (points * mergingPortion) / 100; 
                if (mergingPoints != 0) {
                    points -= mergingPoints;
                    _mergingPoolInstance.addPoints(tokenId, mergingPoints);
                }

                if (points != 0) {
                     /// Add points to the fighter for this round
                    accumulatedPointsPerFighter[tokenId][roundId] += points; 
                    accumulatedPointsPerAddress[fighterOwner][roundId] += points;
                    totalAccumulatedPoints[roundId] += points;
                
                    emit PointsChanged(tokenId, points, true);
                }
            } else {
                /// Do not allow users to reclaim more NRNs than they have at risk
                if (curStakeAtRisk > stakeAtRisk) {
                    curStakeAtRisk = stakeAtRisk;
                }

                /// If the user has stake-at-risk for their fighter, reclaim a portion
                /// Reclaiming stake-at-risk puts the NRN back into their staking pool
                if (curStakeAtRisk != 0) {
                    _stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, tokenId, fighterOwner);
                    amountStaked[tokenId] += curStakeAtRisk;
                }
            }
```
