## [L-01] User will not get information about their `maxId` fighter.

`MergingPool::getFighterPoints` takes argument of `maxId`, which is the maximum token Id of the users fighter.

But since the for loop is implemented like this :
```javascript
for (uint256 i = 0; i < maxId; i++)
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L207

The user will not get information regarding his maxId token and will get information about 0 to `maxId - 1` fighters point.

instead of `i < maxId`, `i <= maxId` should be implemented while for looping.

## [L-02] Two step transfer ownership can be done instead in ALL the contracts for extra safety
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L105-L111

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L86-L92

Also in all remaining contracts wherever transferOwnership is being executed.

## [NC-01] Redundant function for emission of event.
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterOps.sol#L52-L62

A function is made just for emitting an event, not required.