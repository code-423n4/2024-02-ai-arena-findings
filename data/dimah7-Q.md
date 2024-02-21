Severity: Non-critical

Description: 
The `MergingPool::fighterPoints` mapping is used to map to associate points with fighter token ID. 

It's defined as it follows:
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L51

```javascript
/// @notice Mapping of address to fighter points.
    mapping(uint256 => uint256) public fighterPoints;
```

The natspec can cause a confusion since it's saying "address to points", but it's mapping from uint256 to uint256. Based on it's usage in [`MergingPool::pickWinner`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L126-L127) and [`MergingPool::addPoints`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L197) functions, it can be seen that it maps from fighter token ID to points.

Recommendation: 
Consider changing the natspec to "Mapping of fighter ID or fighter owner ID to fighter points." 