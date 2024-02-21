### Low Risk / QA Issues

### [01] Unnecessary emit of PointsAdded event for the Merging Pool when a fight has been won but the stakeAtRisk is more than 0.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L416-L500

```
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
...
}
```

If stakeAtRisk is 0, it means the user is eligible to gain some points based on how much they have staked, but initially points is initialised to 0 within the function  so there is no need to do calculate mergingPoints if points will be 0. It leads to unnecessary execution of `_mergingPoolInstance.addPoints` which under all circumstances emits an pointsAdded event which is state data that doesn't need to be indexed as no points were actually added to the mergingPool.

Recommendation: Wrap the merging points calculation points and addition code within an if statement - `if(points > 0)`.

--

### [02] Total Battles is incorrectly updated twice for the same battle.
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L348

The totalBattles value globally tracks the number of fights that has taken place. A fight between two users is one battle. `RankedBattle.UpdateBattleRecord` is called twice for one fight. Once for the initiator and once for the opponent. The code incorrectly increments total battles by 1 for each call for the respective users in the fight. This would give the indication that two separate battles took place, when it's the same battle. 

Recommendation: Move `totalBattles += 1;` to within the `if (initiatorBool) {` statement and increment the number of battles done globally across the game when the battle record update is sent to the blockchain for the initiator.

--

### [03] GameItems mint allows for quantity to be set to 0 allowing for unnecessary execution and emit of the boughtItem event.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L147-L176

The missing require statement on the quantity allows for unchecked execution of the mint function leading to 0 value transfers and mint and the emitting of the boughtItem event with a quantity - despite no harm to the protocol. It is against best practice for recording unnecessary state data.

Recommendation: add an require statement at the top of the function before main exeuction - `require(amount > 0, "Mint at least one item");`