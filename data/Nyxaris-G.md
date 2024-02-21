## Title: 
Potential Gas Optimization in [`createPhysicalAttributes`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L83) Function.

## Description: 
The  [`createPhysicalAttributes`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L83) function currently uses a dynamic array to store attribute probability indexes. While this does not incur gas costs when called externally due to its view modifier, it is a point of gas inefficiency because  the function is  used internally by state-changing functions.

## Recommendation:
 Consider refactoring the function to use a fixed-size array if the maximum size of the array is known and unchanging. This would preemptively optimize the function for any future internal use within transactions, potentially reducing gas costs.

```javascript
finalAttributeProbabilityIndexes
uint256[6] memory finalAttributeProbabilityIndexes;
```