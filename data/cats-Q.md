# [L-01] `globalStakedAmount` variable is not updated when a player without points loses a fight and the stake is moved from `RankedBattle` to `StakeAtRisk`

When a player loses a battle whilst having staked `$NRN` but has no points accumulated, a portion of their stake is moved from the ranked battle contract to the stake at risk contract. There is a variable that is supposed to track the global staked amount by players in ranked battle, but it is not decreased when the `$NRN` is moved to the other contract.

```solidity
 else if (battleResult == 2) {
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
                accumulatedPointsPerFighter[tokenId][roundId] -= points;
                accumulatedPointsPerAddress[fighterOwner][roundId] -= points;
                totalAccumulatedPoints[roundId] -= points;
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
```

Currently no function uses this variable so it's only low, but this is bad if in the future the variable is used in a function.

# [L-02] Wrong comparison operator in function `Neuron#mint()`
The mint function in the `Neuron.sol` contract makes a wrong check:

```solidity
require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
```

Should be <= MAX_SUPPLY instead of <.