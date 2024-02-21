# [G-1] Unaltered string arrays should be set as fixed-size arrays rather than dynamic-size arrays.

1. `attributes` in `AiArenaHelper.sol` is never altered, and it should therefore be set as a fixed-size array. 
2. Additionally, `defaultAttributeDivisor` in `AiArenaHelper.sol` is never altered, and it should also be set as a fixed-size array.
3. The `probabilities` parameter of the constructor and `addAttributeProbabilities` function of `AiArenaHelper.sol`
4. The `attributeDivisors` parameter of the `addAttributeDivisor` function in `AiArenaHelper.sol`
5. `finalAttributeProbabilityIndexes` array in the `createPhysicalAttributes` of `AiArenaHelper.sol` should be defined in a more optimal way, as well as changed to an array of type uint8's if the values of the array will stay below 256.

## Example relevant code:
```solidity
    string[] public attributes = ["head", "eyes", "mouth", "body", "hands", "feet"];
```

## Recommended Mitigation: 
I recommend the following changes:

1. `string[6] public attributes` instead of `string[] public attributes`
2. `uint8[6] public defaultAttributeDivisor` instead of `uint8[] public defaultAttributeDivisor`
3. `uint8[6][] memory probabilities` instead of `uint8[][] memory probabilities`
4. `uint8[6] calldata attributeDivisors` instead of `uint8[] memory attributeDivisors`
5. `uint8[6] memory finalAttributeProbabilityIndexes;` instead of `uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributes.length);`

**Github:** 
1. https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L17
2. https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L20
3. https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L41, https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131
4. https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L68
5. https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L96


# [G-2] Redundant effects

1. constructor of `AiArenaHelper.sol` calls both `addAttributeProbabilities(0, probabilities);` as well as `attributeProbabilities[0][attributes[i]] = probabilities[i];` while looping through i. Both do the same thing, defining attributeProbabilities[0].
2. 

## Recommended Mitigation:
I recommend the following changes:

1. Remove line 49 of `AiArenaHelper.sol`, getting rid of the redundant effect.


**Github:** 
1. https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L49




# [G-3] Redundant checks

1. If the `attributeDivisors` parameter of `addAttributeDivisor` function of `AiArenaHelper.sol` is set to be of fixed length 6, as G-1 suggest, the check on the parameter's length is unnecessary, since `attributes.length` is always 6.
2. The `transferFrom` call that the `mint` function of `GameItems.sol` makes to `_neuronInstance` does not need to be checked, since that call reverts upon failure anyway.
3. A check in function `claim` of `Neuron.sol` is unnecessary, since the same check is made when the function calls `transferFrom` function.

## Recommended Mitigation:

1. Remove line 70
2. Remove lines 164, 165, and 175 and unindent lines 166-174
3. Remove lines 139-142

**Github:** 
1. https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L70
2. https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L164-L175
3. https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L139-142





# [G-4] Fill arrays with lower size uint's to save storage space.

1. struct `FighterPhysicalAttributes` in `FighterOps.sol` should be changed to have its six objects be uint8's instead of uint256's as long as the objects are designed to be less than 256.

## Recommended Mitigation:

1. `struct FighterPhysicalAttributes {uint8 head; uint8 eyes; uint8 mouth; uint8 body; uint8 hands; uint8 feet;}` instead of struct FighterPhysicalAttributes {uint256 head; uint256 eyes; uint256 mouth; uint256 body; uint256 hands; uint256 feet;}


**Github:** 
1. https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L26-L33




# [G-5] Unnecessary variable usage

1. `uint256 winnerLength` does not need to be defined nor used in the `pickWinner` function of `MergingPool.sol`, since the value it is defined to be (`winners.length`) is already known to be equal to winnerPerPeriod from a prior check in the function on line 120. 

## Recommended Mitigation:
1. Delete line 122 and replace all instances of `winnersLength` with `winnersPerPeriod` in lines 123-131

**Github:** 

1. https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L118-L132



[G-6] More efficient data type could be used

1. Using a `mapping(address => (uint256 => uint8))` to track a user to a round Id to how many wins they had in that round, in `MergingPool.sol` would avoid nested for loops.

### Recommended Mitigation:
1. Define a new mapping of type `mapping(address => (uint256 => uint8))` and index this mapping in functions `claimRewards` and `getUnclaimedRewards` instead of using the `winnerAddresses` mapping. Make the necessary adjustments to alter this new mapping when winners are selected in `pickWinner`.

**Github:** 
1. https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#118-132, https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#139-167, https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#172-185