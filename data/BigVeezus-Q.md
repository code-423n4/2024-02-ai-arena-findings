
## [L-1] `RankedBattle::_addResultPoints` function does not update the `globalStakedAmount` after an ammount has been staked

**Occurences**: 2 times

1. `RankedBattle::_addResultPoints` function at line 465, RankedBattle.sol.
2. `RankedBattle::_addResultPoints` function at line 500, RankedBattle.sol.

**Impact**:
This could cause wrong information to users or stakeholders if the `globalStakedAmount` doesnt update as it should

**Proof Of Code & Added Mitigation**:

```diff
            /// If the user has stake-at-risk for their fighter, reclaim a portion
            /// Reclaiming stake-at-risk puts the NRN back into their staking pool
            if (curStakeAtRisk > 0) {
                _stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, tokenId, fighterOwner);
                amountStaked[tokenId] += curStakeAtRisk;
                // Added code below as solution
+                 globalStakedAmount += curStakeAtRisk;
            }
```

```diff
{
                /// If the fighter does not have any points for this round, NRNs become at risk of being lost
                bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
                if (success) {
                    _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
                    amountStaked[tokenId] -= curStakeAtRisk;
                     // Added code below as solution
+                    globalStakedAmount -= curStakeAtRisk;
                }
            }
```

**globalStakedAmount should be updated to keep proper records of the storage values**
