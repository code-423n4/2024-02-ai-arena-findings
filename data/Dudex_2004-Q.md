Incorrect Event Emission

## Summary 
In the `RankedBattle` contract's `_addResultPoints` function the `PointsChanged` event is not emitting as expected 

## Vulnerability Detail
In the In the `RankedBattle` contract `PointsChanged` event sets bool `true` when the points are added or subtracted from it. But in the `_addResultPoints` function the `PointsChanged` event is not emmiting as expected means when the points are greater than 0 its not setting `PointsChanged` bool as `true`.
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L416-L500

## Impact
Points will not change when user losses

## Tools Used
manual review 

## Recommended Mitigation Steps

```diff

   if (points > 0) {
-           emit PointsChanged(tokenId, points, false);
+           emit PointsChanged(tokenId, points, true);
                }
```