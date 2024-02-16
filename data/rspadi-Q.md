# AI Arena Low Risk and Non-Critical Issues

# Table of Contents
- [Low Risk Findings](#lowrisk)
    - [[L-01] FighterFarm::incrementGeneration : The maximum allowed number of rerolls should be a constant value for each generation](#l01)
    - [[L-02] FighterFarm::claimFighters : Ecrecover is known to be vulnerable to signature malleability](#l02)
    - [[L-03] FighterFarm::redeemMintPass : Validate the length of the iconsTypes array](#l03)
    - [[L-04] MergingPool::claimRewards : Validate the length of the input arrays](#l04)
    - [[L-05] MergingPool::getFighterPoints : Generates array out-of-bounds access](#l05)
    - [[L-06] Neuron::mint : Only MAX_SUPPLY - 1 tokens can be minted](#l06)
    - [[L-07] VoltageManager::spendVoltage : ownerVoltage can underflow](#l07)
    - [[L-08] GameItems::mint : allowanceRemaining may underflow](#l08)
    - [[L-09] RankedBattle::stakeNRN : add tokenId to the Staked and Unstaked events](#l09)
    - [[L-10] RankedBattle::updateBattleRecord : totalBattles state variable is not updated correctly](#l10)
    - [[L-11] RankedBattle::setNewRound : The admin should not have to call two different contract functions in order to synchronize the next roundId](#l11)

- [Informational / Non Critical Findings](#noncritical)
    - [[I-01] AiArenaHelper::addAttributeProbabilities : Don't use hardcoded values](#i01)
    - [[I-02] RankedBattle::_addResultPoints : Don't call addPoints if mergingPoints = 0](#i02)
    - [[I-03] MergingPool::addPoints : Don't execute the function if points is ](#i03)
    - [[I-04] MergingPool : No function to retrieve the points for a specific fighter](#i04)
    - [[I-05] GameItems::createGameItem : Validate input parameters](#i05)     

  
<a id="lowrisk"></a>    
## Low Risk Findings

<a id="l01"></a>
### [L-01] FighterFarm::incrementGeneration : The maximum allowed number of rerolls should be a constant value for each generation

Source: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L132

Currently, the maximum allowed number of rerolls is incremented by one, each time the generation of  afighter type is increased. Initially (for generation 0), the value for maxRerollsAllowed is set to 3 for each fighter type.

Let's assume, at a certain stage after the game has launched, we arrive at generation number 10. A player who is playing the game since generation 0 and who regularly took advantage of rerolls is allowed to perform only one reroll as long as the game remains in generation 10.

A new player, who is just starting out, is allowed to perform 3 + 10 rerolls in generation 10. The difference in terms of allowed rerolls could create some inbalances in the game dynamic.

It would be better to set a fixed level of maximum allowed rerolls for each generation. For example, in generation 0 all users may be allowed to perform 3 rerolls. If the game moves on to generation 1, again all players should be allowed to perform up to 3 rerolls - no matter if they already performed any rerolls in generation 0 or not.


<a id="l02"></a>
### [L-02] FighterFarm::claimFighters : Ecrecover is known to be vulnerable to signature malleability

Source: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L206

The claimFighters() function in the FighterFarm contract calls the verify() function in the Verification contract, which uses the Solidity ecrecover function.

This function is known to be vulnerable to signature malleability and OpenZeppelin's ECDSA helper library should be used instead:

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol 


<a id="l03"></a>
### [L-03] FighterFarm::redeemMintPass : Validate the length of the iconsTypes array 

Source: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L243

The length of the iconsTypes array should be validated at the beginninng of the function. 

Replace:

```
require(mintpassIdsToBurn.length == mintPassDnas.length && mintPassDnas.length == fighterTypes.length && fighterTypes.length == modelHashes.length && modelHashes.length == modelTypes.length);
```

With:

```
require(mintpassIdsToBurn.length == mintPassDnas.length && mintPassDnas.length == fighterTypes.length && fighterTypes.length == iconsTypes.length && iconsTypes.length == modelHashes.length && modelHashes.length == modelTypes.length);
```


<a id="l04"></a>
### [L-04] MergingPool::claimRewards : Validate the length of the input arrays 

Source: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L139 

The length of the modelURIs, modelTypes and customAttributes arrays should be validated at the beginninng of the function.

Add the following code at the beginning of the function:

```
require(modelURIs.length == modelTypes.length && modelTypes.length == customAttributes.length);
```


<a id="l05"></a>
### [L-05] MergingPool::getFighterPoints : Generates array out-of-bounds access

Source: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L206

The size of the points array in the getFighterPoints() view function is hardcoded to the value of 1, but it should be set to maxId.

Make the following code modifications:

```diff
- uint256[] memory points = new uint256[](1); 
+ uint256[] memory points = new uint256[](maxId);
```


<a id="l06"></a>
### [L-06] Neuron::mint : Only MAX_SUPPLY - 1 tokens can be minted 

Source: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L156

Currently, only MAX_SUPPLY - 1 tokens can be minted.

Make the following code modification:

```diff
- require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
+ require(totalSupply() + amount <= MAX_SUPPLY, "Trying to mint more than the max supply");
``` 


<a id="l07"></a>
### [L-07] VoltageManager::spendVoltage : ownerVoltage can underflow

Source: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L110

If the spendVoltage() function is called several times, the value of the ownerVoltage[spender] mapping can underflow.

Add the folling line of code:

```
require(ownerVoltage[spender] >= voltageSpent, "Not enough voltage available");
```

just before the following line:

```
ownerVoltage[spender] -= voltageSpent;
```

The following test can be added to the RankedBattleTest contract to demonstrate this behaviour:

```
function testUpdateBattleRecordVoltageUnderflow() public {
    address player = vm.addr(3);
    _mintFromMergingPool(player);
    _fundUserWith4kNeuronByTreasury(player);

    vm.startPrank(player);
    _rankedBattleContract.stakeNRN(3_000 * 10 ** 18, 0);

    for (uint256 i = 0; i < 11; ++i) {
        _voltageManagerContract.spendVoltage(player, 10);
        console.log("Remaining player voltage: ", uint256(_voltageManagerContract.ownerVoltage(player)));
    }
    vm.stopPrank();
}
```


<a id="l08"></a>
### [L-08] GameItems::mint : allowanceRemaining may underflow

Source: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L169

When calling the mint function, the allowanceRemaining mapping may underflow. This happens, when the following conditions are mey:

The user calls the mint function and specifies a value that is larger than the dailyAllowance for the tokenId to be minted
The dailyAllowanceReplenishTime is smaller than the current timestamp => therefore, allowanceRemaining[msg.sender][tokenId] is not verified 
The remaining code is executed: _neuronInstance.transferFrom... and _replenishDailyAllowance(tokenId)
When the following line is executed, an underflow is generated: allowanceRemaining[msg.sender][tokenId] -= quantity;
  
Provide the following verification at the beginning of the function: 

```
require(quantity <= allGameItemAttributes[tokenId].dailyAllowance, "daily allowance exceeded");
```


<a id="l09"></a>
### [L-09] RankedBattle::stakeNRN : add tokenId to the Staked and Unstaked events

Sources:

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L26
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L29

When emitting the Staked and Unstaked events, the corresponding tokenId should also be specified. Also, the from address and the tokenId should be marked as indexed

Update the Staked and Unstaked events:

```diff
- event Staked(address from, uint256 amount);
- event Unstaked(address from, uint256 amount);

+ event Staked(address indexed from, uint256 indexed tokenId, uint256 amount);
+ event Unstaked(address indexed from, uint256 indexed tokenId, uint256 amount);
```

In the stakeNRN and the unstakeNRN functions, update the emitted event:

```diff
- emit Staked(msg.sender, amount);
+ emit Staked(msg.sender, tokenId, amount);

...

- emit Unstaked(msg.sender, amount);
+ emit Unstaked(msg.sender, tokenId, amount);
```


<a id="l10"></a>
### [L-10] RankedBattle::updateBattleRecord : totalBattles state variable is not updated correctly

Source: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L348

Each time, the game server calls the updateBattleRecord() function, the value for totalBattles is incremented by 1. However, for each battle, there are 2 participants and the updateBattleRecord() function will be called for each participating fighter in order to update the points, stakeAtRisk... 

So, if there are 20 fighters, there should be 10 battles with 2 participants and totalBattles should be set to 10; However, updateBattleRecord() will be called 20 times (for each player) and totalBattles will be set to 20 instead of 10. 

Increment totalBattles only on each second call to updateBattleRecord():

Add the following state variable to the contract:

```
bool private _totalBattlesAlreadyIncremented;
```

At the following code at the end of the updateBattleRecord() function (replace totalBattles += 1;):

```
if(!_totalBattlesAlreadyIncremented) {
    totalBattles += 1;
    _totalBattlesAlreadyIncremented = true;
} 
```

Add the following code to the setNewRound() function, after the require statements:

```
_totalBattlesAlreadyIncremented = false;
``` 


<a id="l11"></a>
### [L-11] RankedBattle::setNewRound : The admin should not have to call two different contract functions in order to synchronize the next roundId 

Source: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L233

Currently, the roundId needs to be set in 3 different contracts and two different contract function calls (performed by an admin) are required to do so. This is dangerous, because the roundId could get out-of-sync in the 3 concerned contracts:

* RankedBattle
* StakeAtRisk
* MergingPool

Ideally, the admin should be required to perform only one function call to start a new round. This should happen in the MergingPool::pickWinner() function.

At the end of a round, the admin calls this function with the array of winners and the pickWinner function should then call the RankedBattle::setNewRound() function, which in turn calls the StakeAtRisk::setNewRound() function.


Add the following interface on top of the MergingPool contract:

```
interface IRankedBattle {
    function setNewRound() external;
}
```

Add the following code at the end of the MergingPool::pickWinner() function:

```
IRankedBattle(_rankedBattleAddress).setNewRound();
```

At the beginning of the RankedBattle::setNewRound() function, make the following modification:

```diff
- require(isAdmin[msg.sender]);
+ require(msg.sender == address(_mergingPoolInstance), "Can only be called by the MergingPool");
```




<a id="noncritical"></a>    
## Informational / Non Critical Findings 

<a id="i01"></a>
### [I-01] AiArenaHelper::addAttributeProbabilities : Don't use hardcoded values

Source: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L133

```diff
- require(probabilities.length == 6, "Invalid number of attribute arrays"); 
+ require(probabilities.length == attributes.length, "Invalid number of attribute arrays");
```


<a id="i02"></a>
### [I-02] RankedBattle::_addResultPoints : Don't call addPoints if mergingPoints = 0

Source: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L451

The addPoints() function onn _mergingPoolInstance should not be called if the value of mergingPoints = 0. 

Provide the following verification:

```
if(mergingPoints > 0)
    _mergingPoolInstance.addPoints(tokenId, mergingPoints);
```


<a id="i03"></a>
### [I-03] MergingPool::addPoints : Don't execute the function if points is 0

Source: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L199

The addPoints() function should not be executed if the value of points is 0.   

Provide the following verification at the beginning of the function:

```
require(points > 0, "No points to be added");
```


<a id="i04"></a>
### [I-04] MergingPool : No function to retrieve the points for a specific fighter

THe MergingPool contains the function getFighterPoints() that allows to list the points for all fighter NFTs up to the specified id. If there are 1000 fighter NFTs and someone would like to retrieve the points for fighter Id #1000, he would need to retrieve a large array of 1000 elements just to get the points for the single fighter he is interested in. 


Therefore, there should also be a function that allows to retrieve the points for a specific fighter id. 

Add the following function to the MergingPool contract:

```
function getSingleFighterPoints(uint256 fighterId) public view returns (uint256) {
    return fighterPoints[fighterId];
}
```

<a id="i05"></a>
### [I-05] GameItems::createGameItem : Validate input parameters

Source: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L208

The following 3 input parameters should be greater than 0 whenever a new game item is created : itemsRemaining, itemPrice and dailyAllowance 

Provide the following verification at the beginning of the function:

```
require(itemsRemaining > 0 && itemPrice > 0 && dailyAllowance > 0, "itemsRemaining, itemPrice and dailyAllowanc should be greater than 0");
```

