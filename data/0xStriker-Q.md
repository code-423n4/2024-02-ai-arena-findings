# Summary



# Low Risk Issues    
no | Issue |Instances||
|-|:-|:-:|:-:|
| [L-01] |  add more validation checks for the input data in the createGameItem() function | 1 |
| [L-02] |  the function call will fail if the maxId is more than one | 1 |
| [L-03] |  zero nrn distribution | 1 |
| [L-04] |  the function adjusts the unstake amount without reverting | 1 |
| [L-05] |  it is a best practice to use the safeErc20 library | 12 |
| [L-06] |  no zero address checks | 15 |

# Non-critical
no | Issue |Instances||
|-|:-|:-:|:-:|
| [N-01] |  The values of the attributeProbabilities mapping are redundantly set| 1 |
| [N-02] |  the function name could be changed to represent it much clearer | 1 |
| [N-03] |  comment suggestion and function implementation difference | 1 |
| [N-04] |  redundant code logic | 1 |
| [N-05] |  the resetting should be skiped if the value is already set to the same value | 3 | 


## [L-01] add more validation checks for the input data in the createGameItem() function 
since the function createGameItem() creates gemeItems that will be in the systme permanently it is a good practice to add some more validation checks for the input data, such as check if the finiteSupply is true, if yes,than the itemsRemaining should be more than zero, and the item price should be more than zero.
```solidity
        if(itemPrice <= 0 ){ 
            revert();
        }
        if (finiteSupply) {
            if(itemsRemaining <= 0 ) {
                revert();
            }
        }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L208-L235

## [l-02] the function call will fail if the maxId is more than one
In the getFighterPoints() function, the array points is initialized with a length of 1, which will cause the function call to fail if maxId is greater than 1. When the loop tries to access points[i] for i > 1, it will result in an out-of-bounds array access, which will cause a runtime error and revert the transaction.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L205-L211

## [L-03] zero nrn distribution
in the setRankedNrnDistribution() function If the rankedNrnDistribution for a round is set to zero (either intentionally or by mistake), players would receive no rewards for that round, which will be unexpected if they have accumulated points. Its best to add a check for zero before adding the distribution value for the round to the rankedNrnDistribution mapping.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L220

## [L-04] the function adjusts the unstake amount without reverting
in the function unstakeNRN there is a check where if the amount to be unstaked is more than the amountStaked it will set the amount to be unstaked to the amountStaked and then continue on unstaking the balance, the problem is it could potentially lead to confusion for users. The function allows a user to attempt to unstake more NRN tokens than they actually have staked, and instead of reverting with an error, it silently adjusts the unstake amount to the maximum available, without informing the user that their input was higher than the their stakedAmount. A user might not realize that they unstaked less than they intended.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L273

## [L-05] it is a best practice to use the safeErc20 library
while the protocol only interacts only with the $NRN token which implements the standard erc20, it is still best practice to use the safERC20 library while interacting with the erc20 functions respectivley (transfer, trnaferFrom, approve) functions.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L376
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L164
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L132
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L143
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L176
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L189
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L203
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L251
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L283
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L493
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L100
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L143

## [L-06] no zero address checks
the functions below does not check for zero address while setting important protocol addresses

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L149
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L157
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L165
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L119
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L141
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L187
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L100
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L95
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L103
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L111
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L120
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L178
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L203
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L211
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L67

## [N-01]  The values of the attributeProbabilities mapping are redundantly set
in the constructor of the AiArenaHelper contract the values for the attributeProbabilities are set twice which is redundant, and should be mitigated to set this value once.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L45
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L49


## [N-02] the function name could be changed to represent it much clearer
the function danToIndex() in the AiArenaHelper contract name could be changed to represent the function much clearer, and since the function does not take dna as input attribute, it could be misleading. The function name could be changed to calculateAttributeIndex() which represents the function's purpose much clearer.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L169

## [N-03] comment suggestion and function implementation difference
the function comment suggests that all the arrays should be of the same length but the require statement doesn't check for the iconsTypes array's length. either the iconsTypes should be added to the checks or the comment should be updated.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L225
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L243

## [N-04] redundant code logic 
in the setStakeAtRiskAddress() function an address is provided to the function to be set as the _stakeAtRiskInstance but the function first assigns the address to the _stakeAtRiskAddress and then assigns the _stakeAtRiskAddress to the _stakeAtRiskInstance. This is redundant.
To fix this redundancy, you can remove the _stakeAtRiskAddress state variable and directly assign the passed address to the function to the _stakeAtRiskInstance while wrapping it in the StakeAtRisk.

the function that includes redundancy
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L192-L196
the redundant state variable
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L69
the place the redundant variable is used which could be replaced with the _stakeAtRiskInstance
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L493

## [N-05]  the resetting should be skiped if the value is already set
after calculating the stakingFactor the _calculatedStakingFactor mapping is set to true for the tokenId and roundId, this mapping should be checked if it already contains the intended value, then the resetting of the mapping to the same value should be skiped. This mapping is used in the stakeNrn and in the unstakeNrn it is possible that one of these functions for the same values are called many times, and since the _calculatedStakingFactor only stores boolean values it is ok to skip the resetting if the values is already the same as the one the contract wants to reset to.
It is the same case with the hasUnstaked mapping in the unstakeNrn function which saves a boolean value. The first time the function is called it will set the value for the tokenId and the roundId to true and if called again this will be reset to the same value which is unnecessary and could be skiped by adding an if statement check.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L262
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L281
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L282