1. [GAS and Duplication]
Inside the AiArenaHelper.sol we see that the attributeProbabilities are set two times. If you look at the constructor, you will see that first time they are set by calling addAttributeProbabilities() and second in the constructor. 

This duplication is unnecessary and can save some gas as well. 

Confirmed in main channel by guppy on 12.02.24 - haha this is a mistake on our part

Code: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L41-L52

2. [Integration risk]

Inside FighterFarm contract, we see that there is a function called setTokenURI, that sets the tokenURI for the given tokenId.

However it is in conflict with the reRoll function:
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L370
When reRoll is called, it resets _tokenURIs[tokenId] = "" which can cause problems.
Consider rewriting this dependancies, because some external sources that depend on the tokenUri can break their functionality hence stop showing the correct uri link.

3. [Integration risk]

In the FighterFarm, GameItems and AAMintPass there is a function that reads the contractUri: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L395,

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L249,

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AAMintPass.sol#L150

This uri link is static and once set, it can't be changed. Consider implementing another function that can change the link if it is not valid anymore. Otherwise the contract will always point to the initial info. 

If you really think that it stays the same, consider implementing it as a storage constant.


4. [Systemic risk]

Overall in the protocol we see the transferOwnership method:
AAMintPass.sol, AiArenaHelper.sol, FighterFarm.sol, GameItems.sol, MergingPool.sol, Neuron.sol, RankedBattle.sol, etc

This function is working correctly, however I suggest using some safer options like Ownable from OpenZeppelin or even Ownable2Step, where some minimal sanity check is implemented. For example not transferring ownership to address(0), which is not included in the current version of the reviewed protocol. 

5. [Systemic risk]

In the Neuron contract we functions like: addMinter, addStaker, addSpender
Those functions add the roles, but there is no way for revoking the roles later. RevokeRole function is not overridden that is why consider this as an option that should be implemented in the future. 

Idea for an attack: The function addSpender allows the spender to steal all of the user allowances. We can argument that the spender is trusted contract, but think about possible update in the future, where some extra functionality is added and we include a new spender who is a malicious account. That will break the entire protocol, because he will be able to steal tokens.

Consider implementing a function where admin can revoke roles again back.

6. [Systemic risk]

In the RankedBattle contract, we see the function called updateBattleRecord: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L322-L328

This function informs the players when they win or lose a game. This function can be potentially frontrun. Consider following possible case:

1: Player stakes Neuron tokens to play.
2: updateBattleRecord is called and player sees that he lost the game by observing the Mempool.
3: Players starts unstaking before updateBattleRecord is recorded, in other words frontrun the result.
4: Now the player is safe because when the result is published he can' t lose his tokens.

This scenario is not really possible on Arbitrum, because the sequencer is forwarding the transactions based on their order, but if the game is deployed on another chain later, that can cause some flaws. 

However in the RPC node on Arbitrum is malicious, potentially this vulnerability can be seen on local node level if the server and the player use same node for connection. 

Posible mitigation is to disable the unstaking feature once starting a game.

7. [Typo misspelling]

In the StakeAtRisk, I think the function updateAtRiskRecords:

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L115,

should be called updateStakeAtRiskRecords, because it is the Stake what is updated.






### Time spent:
30 hours