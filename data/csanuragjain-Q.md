## Fighter can be transferred when staking is active

Link:
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L285C13-L287C14

Description:
Seems like contract only checks whether amountStaked by user is 0. In case user has funds at risk then also he is allowed to transfer fighter

```
if (amountStaked[tokenId] == 0) {
                _fighterFarmInstance.updateFighterStaking(tokenId, false);
            }
```

Recommendation:
User should not be allowed to transfer if user has fund at risk in current round

## Non winner points are carried over

Link:
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L124C9-L128C10

Description:
Seems like only winner points are reset to 0. For non winners the points are not reset and moved to next round. 

Recommendation:
If this is not expected then points for all users should be turned 0 on moving to next round