## The code which does not follow the best practice of `Check-Effects-Interaction`



The code should adhere to the best practice known as *check-effects-interaction*, ensuring that state variables are modified before any *external calls* are executed. This approach effectively mitigates a wide range of *reentrancy vulnerabilities*.

```javascript
File: src/AiArenaHelper.sol

/// @audit The cumulative probability is updated within a loop, following the check-effects-interaction pattern.
179:                  cumProb += attrProbabilities[i];
```
***GitHub***: https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/AiArenaHelper.sol#L179



```javascript
File: src/FighterFarm.sol

/// @audit The 'generation' array is incremented for the specified 'fighterType' in 'incrementGeneration()' before certain assignments.
131:                generation[fighterType] += 1;

/// @audit The 'maxRerollsAllowed' array is incremented for the specified 'fighterType' in 'incrementGeneration()' before certain assignments.
132:                maxRerollsAllowed[fighterType] += 1;

/// @audit The 'nftsClaimed' mapping is updated for the current sender address in 'claimFighters()' before certain assignments.
209:                nftsClaimed[msg.sender][0] += numToMint[0];

/// @audit The 'nftsClaimed' mapping is updated for the current sender address in 'claimFighters()' before certain assignments.
210:                nftsClaimed[msg.sender][1] += numToMint[1];

/// @audit The 'numTrained' mapping is updated for the specified 'tokenId' in 'updateModel()' before certain assignments.
293:                numTrained[tokenId] += 1;

/// @audit The 'totalNumTrained' variable is incremented in 'updateModel()' before certain assignments.
294:                totalNumTrained += 1;

```

***GitHub***: https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/FighterFarm.sol#L131,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/FighterFarm.sol#L132,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/FighterFarm.sol#L209,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/FighterFarm.sol#L210,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/FighterFarm.sol#L293,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/FighterFarm.sol#L294

```javascript
File: src/GameItems.sol

/// @audit Multiple external calls are made before this assignment, which precedes the best practice of check-effects-interaction pattern.
234:             _itemCount += 1;
```

***GitHub***: https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/GameItems.sol#L234


```javascript
File: src/MergingPool.sol

/// @audit The number of rounds claimed for the message sender is incremented by 1, potentially indicating a new round claimed.
150:         numRoundsClaimed[msg.sender] += 1;

/// @audit The variable `claimIndex` is incremented by 1, potentially indicating progression in claiming rewards.
160:         claimIndex += 1;

/// @audit The number of rewards is incremented by 1, potentially indicating addition of a new reward.
180:         numRewards += 1;

/// @audit The points for the fighter corresponding to `tokenId` are incremented by `points`, potentially indicating an increase in points.
197:         fighterPoints[tokenId] += points;

/// @audit The total points in the merging pool is incremented by `points`, potentially indicating an increase in total points.
198:         totalPoints += points;

```

***GitHub***: https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/MergingPool.sol#L150,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/MergingPool.sol#L160,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/MergingPool.sol#L180,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/MergingPool.sol#L197,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/MergingPool.sol#L198



```javascript
File: src/RankedBattle.sol

/// @audit The variable `roundId` is incremented by 1 here, potentially setting the round ID for the next round.
236:         roundId += 1;

/// @audit The number of rounds claimed for the message sender is incremented by 1, potentially indicating a new round claimed.
304:         numRoundsClaimed[msg.sender] += 1;

/// @audit The amount of NRN tokens claimed for the message sender is incremented by `claimableNRN`, indicating claiming of rewards.
307:         amountClaimed[msg.sender] += claimableNRN;

/// @audit The variable `points` is decreased by `mergingPoints`, potentially indicating subtraction of points due to merging.
450:         points -= mergingPoints;

/// @audit The number of wins for the fighter corresponding to `tokenId` is incremented by 1, potentially indicating a win in a battle.
507:         fighterBattleRecord[tokenId].wins += 1;

/// @audit The number of ties for the fighter corresponding to `tokenId` is incremented by 1, potentially indicating a tie in a battle.
509:         fighterBattleRecord[tokenId].ties += 1;

/// @audit The number of losses for the fighter corresponding to `tokenId` is incremented by 1, potentially indicating a loss in a battle.
511:         fighterBattleRecord[tokenId].loses += 1;
```

***GitHub***: https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/RankedBattle.sol#L236,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/RankedBattle.sol#L304,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/RankedBattle.sol#L307,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/RankedBattle.sol#L450,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/RankedBattle.sol#L507,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/RankedBattle.sol#L509,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/RankedBattle.sol#L511



```javascript
File: src/StakeAtRisk.sol
 
/// @audit The state variables are updated within a loop, following the check-effects-interaction pattern.

123            stakeAtRisk[roundId][fighterId] += nrnToPlaceAtRisk;
124            totalStakeAtRisk[roundId] += nrnToPlaceAtRisk;
125            amountLost[fighterOwner] += nrnToPlaceAtRisk;
```

***GitHub***: https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/StakeAtRisk.sol#L123,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/StakeAtRisk.sol#L124,
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/StakeAtRisk.sol#L125


```javascript
File: src/VoltageManager.sol

/// @audit Assignment follows a conditional check, which precedes the best practice of check-effects-interaction pattern.
110:            ownerVoltage[spender] -= voltageSpent;

```
***GitHub***:
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/VoltageManager.sol#L110