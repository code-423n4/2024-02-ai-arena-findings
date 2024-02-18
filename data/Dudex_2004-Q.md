Incorrect Event Emission

## Summary 
In the `RankedBattle` contract in the `_addResultPoints` function the `PointsChanged` event is not emitting as expected 

## Vulnerability Detail
In the In the `RankedBattle` contract `PointsChanged` event sets bool `true` when the points are added or subtracted from it. But in the `_addResultPoints` function the `PointsChanged` event is not emmiting as expected means when the points are greater than 0 its not setting `PointsChanged` bool as `true`.


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