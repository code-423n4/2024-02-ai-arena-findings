#### [L01] addAttributeProbabilities should be called when generation is incremented. (otherwise reroll may have no stats)
###### Summary:
When FighterFarm#incrementGeneration() is called both the generation and the maxRerollsAllowed are incremented. However if a user decides to utilise this reroll before an admin has the chance to call addAttributeProbabilities for the new generation then the call to AiArenaHelper#getAttributeProbabilities will return the default values of 0.
###### LoC:
[FighterFarm.sol#L129-L134](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L129-L134) 
[FighterFarm.sol#L383-L388](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L383-L388) 
[AiArenaHelper.sol#L131-L139](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L131-L139)
###### Recommendation:
I recommend adding a call to AiArenaHelper#addAttributeProbabilities in FighterFarm#incrementGeneration to update the attribute the probabilities for the new generation at the same time it is created.
```solidity
function incrementGeneration(uint8 fighterType, uint8[][] memory probabilities)) external returns (uint8) {
	require(msg.sender == _ownerAddress);
	generation[fighterType] += 1; 
	_aiArenaHelperInstance.addAttributeProbabilities(generation[fighterType], probabilities)
	maxRerollsAllowed[fighterType] += 1;
	return generation[fighterType];
}
```

#### [L02] claimFighters is vulnerable to cross chain replay attacks due to no chain.id in the msgHash.
###### Summary:
If AI Arena is only ever deployed on Arbitrum this is a non issue, however if the protocol team ever decides to deploy on other networks as well users will be able to user the Arbitrum signatures to claim a bunch of fighters instantly.
###### LoC:
[FighterFarm.sol#L199-L205](https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/FighterFarm.sol#L199-L205)
###### Recommendation:
I recommend adding a chain.id to the message hash to allow the protocol to be deployed on to multiple networks in the future without needing to be modified.

#### [L03] Batteries can be used when replenishes are available.
###### Summary:
Currently when users call useVoltageBattery it is possible for users to waste items when they may have replenishes available.
###### LoC:
[VoltageManager.sol#L93-L99](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L93-L99) 
###### Recommendation:
I recommend adding a check of ownerVoltageReplenishTime[spender] <= block.timestamp and calling replenishVoltage if it returns true to prevent users from wasting voltageBatteries when they have a replenish available.


#### [N01] FighterOps#viewFighterInfo() returns incorrect uint type.
###### Summary:
The viewFighterInfo function currently returns self.generation as a uint16 when the Fighter#generation is declared as a uint8. While it wont cause any issues i 
###### LoC:
[FighterOps.sol#L102](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterOps.sol#L102) 
###### Recommendation:
Update [line 92](https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/FighterOps.sol#L92) to uint8.








