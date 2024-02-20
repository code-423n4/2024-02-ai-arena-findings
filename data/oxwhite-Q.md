##L1:Inconsistent type usage for "generation"
in AiArenaHelper.sol inconsistent type is used for fighter generation.It can be seen from the mapping given below that  uint256 is set to it. 
```solidity
    /// @notice Mapping tracking fighter generation to attribute probabilities
    mapping(uint256 => mapping(string => uint8[])) public attributeProbabilities;
```
However in the function [deleteAttributeProbabilities](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L144) the type is used for generation is uint8. 
```solidity
        function deleteAttributeProbabilities(uint8 generation) public {
        require(msg.sender == _ownerAddress);


        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = new uint8[](0);
        }
    }
```

In the functions [getAttributeProbabilities](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L157) and [dnaToIndex](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L169) again uint256 is used for "generation" param.That causes confusion and it is not recommended for overall code clearity.

##L2:No Event for important changes

Events are important when it comes to updating state of the contracts. It is recommended to emit an event that shows the old value and the new value in the following functions :
1. The function [transferOwnership](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L120) is defined in all the contracts mentioned below. 
```
FighterFarm.sol
GameItems.sol,
MerginPool.sol,
Neuron.sol,
RankedBattle.sol,
VoltageManager.sol,
AiArenaHelper.sol,

```
2.In MergingPool.sol the function *updateWinnersPerPeriod()*,
```solidity
   function updateWinnersPerPeriod(uint256 newWinnersPerPeriodAmount) external {
        require(isAdmin[msg.sender]);
        winnersPerPeriod = newWinnersPerPeriodAmount;
    }     
```
3.In RankedBattle.sol the function *setRankedNrnDistribution*,
```solidity
  function setRankedNrnDistribution(uint256 newDistribution) external {
        require(isAdmin[msg.sender]);
        rankedNrnDistribution[roundId] = newDistribution * 10**18;
    } 
```
4.In FighterFarm.sol the function *addStaker*,
```solidity
  function addStaker(address newStaker) external {
        require(msg.sender == _ownerAddress);
        hasStakerRole[newStaker] = true;
    } 
```



