## [L-01] User will not get information about his `maxId` fighter in `MergingPool::getFighterPoints` function , because the function takes argument of `maxId`, which is the maximum token Id of the users fighter.

But since the for loop is implemented like this :
```javascript
for (uint256 i = 0; i < maxId; i++)
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L207

The user will not get information regarding his maxId token and will get information about 0 to `maxId - 1` fighters point.

instead of `i < maxId`, `i <= maxId` should be implemented while for looping.