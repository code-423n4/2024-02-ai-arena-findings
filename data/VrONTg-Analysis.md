
1.	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L150 
	require(_neuronInstance.balanceOf(msg.sender) >= price, "Not enough NRN for purchase");
 	this check redundant, because further we do it in ERC20 SC 
	
	Recommended Mitigation Steps
	delete line
	require(_neuronInstance.balanceOf(msg.sender) >= price, "Not enough NRN for purchase");   
	
2.	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L194
	setTokenURI function  must have strict input for impossible set tokenURI for non existing items/tokens.
	
	Recommended Mitigation Steps
	require(_tokenID < _itemCount);

3.	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L151 
	here we can see redundant check, because no need check  allGameItemAttributes[tokenId].finiteSupply == true after ||
	
	Recommended Mitigation Steps
	
	REPLACE:
	require(
            allGameItemAttributes[tokenId].finiteSupply == false || 
            (
                allGameItemAttributes[tokenId].finiteSupply == true && 
                quantity <= allGameItemAttributes[tokenId].itemsRemaining
            )
        );

	FOR:
 
	require(!tempGameItemAttributes.finiteSupply ||
            quantity <= tempGameItemAttributes.itemsRemaining);
	
	
4. 	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L147
	function mint has implicit logic for spend Neuron tokens. It's not good for users. They must know how much tokens they spend if execute this function of SC.
	
	Recommended Mitigation Steps
	add input field for tokens amount.


5. 	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L95
	redundant check in  useVoltageBattery() function -  require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0); due 
	further  check in ERC1155 standart require(fromBalance >= amount, "ERC1155: burn amount exceeds balance");

	Recommended Mitigation Steps
	delete 
	require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);

6. 	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L105
	function spendVoltage(address spender, uint8 voltageSpent) - if user send here 255 as voltageSpent argument, the function will implicitly return an error due 
	this line  -  ownerVoltage[spender] -= voltageSpent; because ownerVoltage[spender] can'not return value more than 100.

	Recommended Mitigation Steps
	add check
	require(voltageSpent<=100, "wrong value!");

7. 	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L111
	no need read info from state emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);

	Recommended Mitigation Steps
	just replace it for 100
      	emit VoltageRemaining(msg.sender, 100);

8. 	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L41
	Repeated logic in constructor. There is no point do assignment of atributes and probabilities twice.

	Recommended Mitigation Steps
	delete function  addAttributeProbabilities() call from constructor 

	
        constructor(uint8[][] memory probabilities) {
        	_ownerAddress = msg.sender;
        	uint256 attributesLength = attributes.length;
        	for (uint8 i = 0; i < attributesLength; i++) {
            		attributeProbabilities[0][attributes[i]] = probabilities[i];
            		attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        	}
    	} 
	
9. 	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L83
	through dna user can predict traits of his fighter.

	Recommended Mitigation Steps
	Use Chainlink oracle ro fetch random numbers.

10.	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AAMintPass.sol#L125
	don't use EIP712, so all signature doesn't verify address of SC, chain ID, version of SC.

	Recommended Mitigation Steps
	Use EIP712 mechanism for verify signature.

11.	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AAMintPass.sol#L72
	In function remove Admin we must check is it true admin in isAdmin mapping

	Recommended Mitigation Steps
	add check 
	require(isAdmin[adminAddress], "not admin");

12.	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AAMintPass.sol#L123
	_tokenURIs array doesn't packed properly in claimMintPass function. 

	Recommended Mitigation Steps
	add proper packing.
	abi.encodePacked(_tokenURIs)

13.	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L373
	require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll"); in reRoll() function redundant check,  because further we do it in ERC20 SC
	
	Recommended Mitigation Steps
	delete
	require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");

14.	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L121
	require(!isSelectionComplete[roundId], "Winners are already selected"); in function pickWinner() it's redundant check, because 
	roundID increments only in this same function, so it can't be executed with same roundID never.

	Recommended Mitigation Steps
	delete 
	require(!isSelectionComplete[roundId], "Winners are already selected");

15.	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L139C14-L139C26
	there is no check for proper length of arrays inputs in claimRewards function. This can lead to implicit revert function.
	
	Recommended Mitigation Steps
	add check for proper arrays length	
		
16.	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L308
	claimNRN() function can mint tokens to smart contract which are not support ERC20 tokens and assets can be frozen there.
	
	Recommended Mitigation Steps
	Use SafeMint() instead mint()
		
17.	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L247
	require(_neuronInstance.balanceOf(msg.sender) >= amount, "Stake amount exceeds balance"); -  redundant check, because further
	in ERC20 when execute transferFrom we have this check.

	Recommended Mitigation Steps
	delete redundant check.

18.	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L246 
	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L271
	require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter"); this requirement means that 
	no one operator can manage trusted to him NFTs, only owner. It makes this SC not ERC721 compatible.

	Recommended Mitigation Steps
	use require(_isApprovedOrOwner(_msgSender(), tokenId), "ERC721: caller is not token owner nor approved"); instead

19.	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L341
	In function updateBattleRecord  we execute external call to _stakeAtRiskInstance
	and retrieve uint256 stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
	but further in _addResultPoints private function 
	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L430
	we do same call. Don't do this.
	_stakeAtRiskInstance.getStakeAtRisk(tokenId); this value can't change anywhere except
	within 	_addResultPoints private function
	
	Recommended Mitigation Steps
	Just send fetch data to  _addResultPoints private function as argument.
	_addResultPoints(battleResult, tokenId, eloFactor, mergingPortion, fighterOwner, stakeAtRisk);

20.	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L449C13-L451C21
	_addResultPoints function lines of code about merging points and fighterpoints https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L466C12-L466C18
	can execute properly only if stakeAtRisk == 0, otherwise points will be equal to 0 and execute all this code no sense.
	 if (stakeAtRisk == 0) { 
                points = stakingFactor[tokenId] * eloFactor; 
            }
	
	Recommended Mitigation Steps
	Put lines of code about merging points and fighterpoints within  if (stakeAtRisk == 0) 
	Moreover. You can replace this lines 
 
	
	if (points > 0) {
                emit PointsChanged(tokenId, points, true);
        } 
	inside "if (stakeAtRisk == 0)" too.

	and replace if (points > 0) this check for require(eloFactor!=0). 
	In this case points will be always more than 0 and we can delete "if (points > 0)" this check for emit event.


21.	treasuryAddress			https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L58C20-L58C35
	traits (price, quantity)  	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L40 
	attributeToDnaDivisors 		https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L33C37-L33C58
	treasuryAddress 		https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L48
	_delegatedAddress 		https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L54
	rerollCost   			https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L39C20-L39C31
	_rankedBattleAddress		https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L33	
	_rankedBattleAddress 		https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L38
	_fighterFarmInstance   		https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L41C17-L41C37
	_fighterFarmInstance		https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L85C17-L85C37	
	_voltageManagerInstance  	https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L88C20-L88C43

	there is no function for change this values from above. It's not good if owner want change rerollCost for example and they must redeploy all SC.
	otherwise maybe it's logic of project. I don't know.
	
	Recommended Mitigation Steps
	add function for change these values.

### Time spent:
50 hours