# Quality Assurance

| ID     | Issues                                                                                                                                    |
|--------|-------------------------------------------------------------------------------------------------------------------------------------------|
| [L-01] | 0 value function calls can spam off-chain tracking system                                                                                 |
| [L-02] | Function spendVoltage() only allows spending 255 voltage maximum                                                                          |
| [L-03] | Avoid hardcoding tokenId 0 for battery game item                                                                                          |
| [L-04] | Owner address is not provided admin access on transferOwnership()                                                                         |
| [L-05] | Admin access of previous owner is not revoked when ownership is transferred                                                               |
| [L-06] | Function updateFighterStaking() is not updated if user loses all his stake and round ends                                                 |
| [L-07] | Incorrect use of < instead of <= allows minting maximum of 1 NRN token less than expected total supply of 1 billion                       |
| [L-08] | Consider pausing/access controlling function burn() initially for a guarded launch                                                        |
| [L-09] | Transferring/selling Fighter NFT before winners are picked causes loss of reward NFT to previous owner                                    |
| [L-10] | Function claimRewards() does not check if the custom attribute values for weight and element fall in range [65,95] and [0,2] respectively |
| [L-11] | Nested for loops in function claimRewards() can DOS due to OOG exception                                                                  |
| [L-12] | Attacker can reroll the Fighter NFT before buyer buys it on a marketplace                                                                 |
| [L-13] | Multiple problems with getFighterPoints() causes admin to be unable to pick winners directly                                              |
| [L-14] | ID substitution mechanism is not implemented for tokenIds having customURI length equal to 0                                              |
| [L-15] | Attacker can artificially inflate numTrained for his fighterType                                                                          |
| [L-16] | The owner of a tokenId can use up all rerolls before selling it on a marketplace                                                          |
| [L-17] | Attacker can intentionally DOS user's max capacity by sending fighters from multiple accounts created                                     |
| [L-18] | Consider spreading out probabilities for iconTypes                                                                                        |
| [L-19] | Ensure sum of all probabilities of a specific attribute is always 100                                                                     |
| [N-01] | No need to initialize roundId to 0                                                                                                        |
| [N-02] | Consider using msg.sender instead of passing _ownerAddress as parameter in the constructor                                                |
| [N-03] | Missing event emission when transferring ownership                                                                                        |
| [N-04] | OZ recommends using _grantRole() instead of deprecated _setupRole() function                                                              |
| [N-05] | Do not decrease allowance if allowance is set to type(uint256).max                                                                        |
| [N-06] | ncorrect logical natspec comment should be corrected                                                                                      |
| [N-07] | Consider passing bool as value to function setAllowedBurningAddresses()                                                                   |
| [N-08] | setTokenURI() function does not check if tokenId exists                                                                                   |
| [N-09] | Do not hardcode string in function contractURI()                                                                                          |
| [N-10] | In 83 years, typecast to uint32 would break in function _replenishDailyAllowance()                                                        |
| [N-11] | Function viewFighterInfo() does not return every data of the fighter                                                                      |
| [N-12] | Reroll cost of 1000 NRN cannot be changed in the future                                                                                   |
| [N-13] | There can only be 255 generations following which incrementGeneration() would revert                                                      |
| [N-14] | Do not hardcode generation to 0 in constructor                                                                                            |

## [L-01] 0 value function calls can spam off-chain tracking system

It would be cheap (due to Arbitrum chain) to make 0 value spendVoltage() calls to spam the off-chain monitoring system or game server with event emissions. 
```solidity
File: VoltageManager.sol
109:     function spendVoltage(address spender, uint8 voltageSpent) public {
110:         require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
111:         if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
112:             _replenishVoltage(spender);
113:         }
114:         ownerVoltage[spender] -= voltageSpent;
115:         emit VoltageRemaining(spender, ownerVoltage[spender]);
116:     }
```

Another instance:
claim() function calls can spam 0 amount calls.
```solidity
File: Neuron.sol
147:     function claim(uint256 amount) external {
148:         
149:         require(
150:             allowance(treasuryAddress, msg.sender) >= amount, 
151:             "ERC20: claim amount exceeds allowance"
152:         );
153:         
154:         transferFrom(treasuryAddress, msg.sender, amount);
155:         emit TokensClaimed(msg.sender, amount);
156:     }
157: 
```

Another instance:

Anyone can call this function to spam the offchain system. 
```solidity
File: FighterOps.sol
54:     function fighterCreatedEmitter(
55:         uint256 id,
56:         uint256 weight,
57:         uint256 element,
58:         uint8 generation
59:     ) 
60:         public 
61:     {
62:         emit FighterCreated(id, weight, element, generation);
63:     }
```

## [L-02] Function spendVoltage() only allows spending 255 voltage maximum

The parameter voltageSpent only allows type(uint8).max i.e. 255 as the max voltage that can be spent. If AIArena integrates more games in the future that use voltage > 255, this would be an issue since this contract would not be able to handle the input parameter values.
```solidity
File: VoltageManager.sol
109:     function spendVoltage(address spender, uint8 voltageSpent) public {
110:         require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
111:         if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
112:             _replenishVoltage(spender);
113:         }
114:         ownerVoltage[spender] -= voltageSpent;
115:         emit VoltageRemaining(spender, ownerVoltage[spender]);
116:     }
```

## [L-03] Avoid hardcoding tokenId 0 for battery game item

TokenId 0 is hardcoded for the voltage battery game item. This would be an issue if the first gate item created is not a battery. Consider using it a setter function to set the tokenId value or save it as a constant. 
```solidity
File: VoltageManager.sol
094:     function useVoltageBattery() public {
095:         require(ownerVoltage[msg.sender] < 100);
096:         require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0); 
097:         _gameItemsContractInstance.burn(msg.sender, 0, 1);
098:         ownerVoltage[msg.sender] = 100;
099:         emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
100:     }
```

## [L-04] Owner address is not provided admin access on transferOwnership()

In the constructor, when the owner address is initially assigned, it is also provided admin access. It would be convenient to also provide admin access to the new owner when the ownership is transferred below.
```solidity
File: RankedBattle.sol
167:     function transferOwnership(address newOwnerAddress) external {
168:         require(msg.sender == _ownerAddress);
169:         _ownerAddress = newOwnerAddress;
171:     }
```

## [L-05] Admin access of previous owner is not revoked when ownership is transferred

When the ownership is transferred from one address to another, the admin access of the previous owner is not revoked. This could lead to unnoticed problems. Since the new owner can revoke the access of the old owner at any time, this issue has been submitted as low-severity.
```solidity
File: RankedBattle.sol
167:     function transferOwnership(address newOwnerAddress) external {
168:         require(msg.sender == _ownerAddress);
169:         _ownerAddress = newOwnerAddress;
171:     }
```

## [L-06] Function updateFighterStaking() is not updated if user loses all his stake and round ends

The issue is that if the user loses all his staked NRN to the stakeAtRisk contract and the round ends, the user's fighter NFT's updateFighterStaking() is not updated to false. This would prevent the user from selling/transferring his NFT even though `amountStaked` = 0. The issue can be solved by calling unstakeNRN() which would solve this issue.

I was initially confused about the severity of this issue but low-severity in my opinion serves this issue best since no funds are at risk and if the issue was found in production, the team could have figured out it easily i.e. calling unstakeNRN().
```solidity
File: RankedBattle.sol
496:             } else {
497:                 /// If the fighter does not have any points for this round, NRNs become at risk of being lost
498:                 bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
499:                 if (success) {
500:                     _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
501:                     amountStaked[tokenId] -= curStakeAtRisk;
502:                  
503:                 }
```

## [L-07] Incorrect use of < instead of <= allows minting maximum of 1 NRN token less than expected total supply of 1 billion

Currently, the total mintable supply is expected to be 1 billion. The code currently though only allows minting maximum of 1 billion tokens - 1 token of NRN. Use <= instead of < to meet intended spec.
```solidity
File: Neuron.sol
166:     function mint(address to, uint256 amount) public virtual {
167:         require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
```

## [L-08] Consider pausing/access controlling function burn() initially for a guarded launch

Currently, the Neuron contract allows anyone to burn their tokens. The issue is that although this might be helpful in the future if more games use burning for some game logic, leaving it publicly accessible at launch allows whales or any NRN token holder to manipulate the totalSupply. 

Solution: Consider pausing or access controlling function burn() initially to ensure a guarded launch and preventing mass NRN burning. 
```solidity
File: Neuron.sol
173:     function burn(uint256 amount) public virtual {
174:         _burn(msg.sender, amount);
175:     }
```

## [L-09] Transferring/selling Fighter NFT before winners are picked causes loss of reward NFT to previous owner

If the user transfers his fighter NFT before the winners are picked for that round, then the previous owner would lose the reward NFT if the tokenId won. This technically falls under user mistake but I felt it was worth pointing out. 
```solidity
File: MergingPool.sol
119:     function pickWinner(uint256[] calldata winners) external {
120:         require(isAdmin[msg.sender]);
121:         require(winners.length == winnersPerPeriod, "Incorrect number of winners");
122:         require(!isSelectionComplete[roundId], "Winners are already selected"); 
123:         uint256 winnersLength = winners.length;
124:         address[] memory currentWinnerAddresses = new address[](winnersLength);
125:         for (uint256 i = 0; i < winnersLength; i++) {
126:            
127:             currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);
128:             totalPoints -= fighterPoints[winners[i]];
129:             fighterPoints[winners[i]] = 0; 
130:         }
131:         winnerAddresses[roundId] = currentWinnerAddresses; 
132:         isSelectionComplete[roundId] = true;
133:         roundId += 1;
134:     }
```

## [L-10] Function claimRewards() does not check if the custom attribute values for weight and element fall in range [65,95] and [0,2] respectively

The function claimRewards() allows users to provide customAttributes to mint a fighter NFT with any weight and element they want. But it does not check if the values fall in the range [65,95] and [0,2] respectively. 
```solidity
File: MergingPool.sol
143:     function claimRewards(
144:         string[] calldata modelURIs, 
145:         string[] calldata modelTypes,
146:         uint256[2][] calldata customAttributes
147:     ) 
148:         external 
149:     {
150:         uint256 winnersLength;
151:         uint32 claimIndex = 0;
152:         uint32 lowerBound = numRoundsClaimed[msg.sender];
153:         for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
154:             numRoundsClaimed[msg.sender] += 1;
155:             winnersLength = winnerAddresses[currentRound].length;
156:             for (uint32 j = 0; j < winnersLength; j++) {
157:                 if (msg.sender == winnerAddresses[currentRound][j]) {
158:                     _fighterFarmInstance.mintFromMergingPool( 
159:                         msg.sender,
160:                         modelURIs[claimIndex],
161:                         modelTypes[claimIndex],
162:                         customAttributes[claimIndex]
163:                     );
164:                     claimIndex += 1;
165:                 }
166:             }
167:         }
168:         if (claimIndex > 0) {
169:             emit Claimed(msg.sender, claimIndex);
170:         }
171:     }
```

## [L-11] Nested for loops in function claimRewards() can DOS due to OOG exception

Impact: Claiming can DOS for buyers with roundId too far away from base index of 0 and due to the use of nested for loops.

Coded POC:
 - Run the test using `forge test --mt testOOGGasIssue -vvvvv` to see the gas used by claimRewards(). 
 - In the scenario described in the POC, it takes around 1.1M gas (1122817) if a user one year in the future decides to use AiArena. This gas cost is higher than the expected cost of 300k.
 - Due to this, the claim calls for that user would always revert with an OOG error preventing the user from claiming his Fighter NFTs.
```solidity
File: MergingPool.t.sol
354:     function testOOGGasIssue() public {
355:         uint256[] memory winnersArray = new uint256[](50); // Just includes alice many times
356: 
357:         //Mints tokens
358:         for (uint256 i=1; i <= 50 ; i++) {
359:             vm.prank(address(_mergingPoolContract));
360:             _fighterFarmContract.mintFromMergingPool(vm.addr(i), "_neuralNetHash", "original", [uint256(1), uint256(80)]);
361:             winnersArray[i-1] = i-1;
362:         }
363: 
364:         //Skip 24 rounds (i.e. a gamer starts competing in ai arena after a year)
365:         //Assume 50 winners per round since rounds run for 2 weeks
366:         for (uint256 i; i < 24; i++) {
367:             _mergingPoolContract.updateWinnersPerPeriod(50);
368:             _mergingPoolContract.pickWinner(winnersArray);
369:         }
370: 
371:         // Mint a token to bob first
372:         vm.prank(address(_mergingPoolContract));
373:         _fighterFarmContract.mintFromMergingPool(makeAddr("bob"), "_neuralNetHash", "original", [uint256(1), uint256(80)]);
374: 
375:         // Bob wins on round 24
376:         //uint256[] memory newWinnerBob = new uint256[](1);
377:         //newWinnerBob[0] = 50;
378:         winnersArray[25] = 50;
379:         _mergingPoolContract.updateWinnersPerPeriod(50);
380:         _mergingPoolContract.pickWinner(winnersArray);
381: 
382:         // Bob calls claimRewards()
383:         testClaimingIssue();
384:     }
385: 
386:     function testClaimingIssue() public {
387:         string[] memory _modelURIs = new string[](2);
388:         _modelURIs[0] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
389:         string[] memory _modelTypes = new string[](2);
390:         _modelTypes[0] = "original";
391:         uint256[2][] memory _customAttributes = new uint256[2][](2);
392:         _customAttributes[0][1] = uint256(1);
393:         _customAttributes[0][0] = uint256(80);
394:         vm.prank(makeAddr("bob"));
395:         _mergingPoolContract.claimRewards(_modelURIs, _modelTypes, _customAttributes);
396:     }
```

Test logs (see gas amount used at top of the log):
```solidity
   ├─ [1122817] MergingPool::claimRewards(["ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m", ""], ["original", ""], [[80, 1], [0, 0]])
    │   ├─ [380867] FighterFarm::mintFromMergingPool(bob: [0x1D96F2f6BeF1202E4Ce1Ff6Dad0c2CB002861d3e], "", "original", [80, 1])
    │   │   ├─ [32787] AiArenaHelper::createPhysicalAttributes(66610918247435599241959816232774984802237534394899613124736515487129290140817 [6.661e76], 0, 0, false) [staticcall]
    │   │   │   └─ ← FighterPhysicalAttributes({ head: 1, eyes: 1, mouth: 3, body: 2, hands: 2, feet: 2 })
    │   │   ├─ emit Transfer(from: 0x0000000000000000000000000000000000000000, to: bob: [0x1D96F2f6BeF1202E4Ce1Ff6Dad0c2CB002861d3e], tokenId: 51)
    │   │   ├─ [2260] FighterOps::d8d02d30(0000000000000000000000000000000000000000000000000000000000000033000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000500000000000000000000000000000000000000000000000000000000000000000) [delegatecall]
    │   │   │   ├─ emit FighterCreated(id: 51, weight: 1, element: 80, generation: 0)
    │   │   │   └─ ← ()
```

Solution: Consider implementing the claimRewards() function again but this time allowing the user to claim for specific rounds they won in. The specific rounds could be stored in a mapping, which the user could retrieve using a getter function.

## [L-12] Attacker can reroll the Fighter NFT before buyer buys it on a marketplace

The reRoll() function allows users to reroll their NFTs with random traits. The issue is that if an attacker has listed their NFT on a marketplace and a buyer likes it for the traits it currently has, the attacker can call reRoll() on seeing the buyer's offer before selling it to him. Due to this, the buyer won't receive the Fighter NFT with the attributes, element and weight it originally came with, causing harm to him and his purchase. 
```solidity
    /// @notice Rolls a new fighter with random traits.
    /// @param tokenId ID of the fighter being re-rolled.
    /// @param fighterType The fighter type.
    function reRoll(uint8 tokenId, uint8 fighterType) public {
        require(msg.sender == ownerOf(tokenId));
        require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
        require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");
```

## [L-13] Multiple problems with getFighterPoints() causes admin to be unable to pick winners directly

There are three problems with this array:
1. Line 210 declares the array with a fixed size of 1 even though the for loop expects it to be used dynamically.
2. Line 211 has the check i < maxId. This is incorrect since the natspec above the function expects it run upto the maxId. Thus, it should use <=.
3. The for loop would DOS in the future due maxId being the tokenId value, which could be extremely large if alot of fighters are minted over time.

The consequence is that the admin is not able to pick winners for the round since the admin cannot access the fighterPoints. This problem is temporary though since  the frontend could just run the same for loop and access the figherPoints from the publicly accessible mapping. 
```solidity
File: MergingPool.sol
209:     function getFighterPoints(uint256 maxId) public view returns(uint256[] memory) {
210:         uint256[] memory points = new uint256[](1);
211:         for (uint256 i = 0; i < maxId; i++) {
212:             points[i] = fighterPoints[i];
213:         }
214:         return points;
215:     }
```

## [L-14] ID substitution mechanism is not implemented for tokenIds having customURI length equal to 0

If the tokenId has customURI length = 0, then the uri() function returns the baseURI set during deployment. The issue is that [ERC1155 Metadata specification](https://eips.ethereum.org/EIPS/eip-1155#metadata) requires it to be a MUST to append tokenId's to the baseURI if ids exist for the URI. Since there is no custom URI for the tokenId, this substitution should be done as specified.  
```solidity
File: GameItems.sol
262:     function uri(uint256 tokenId) public view override returns (string memory) {
263:         string memory customURI = _tokenURIs[tokenId];
264:         if (bytes(customURI).length > 0) {
265:             return customURI;
266:         }
267:         return super.uri(tokenId);
268:     }        
```

## [L-15] Attacker can artificially inflate numTrained for his fighterType

The owner of a tokenId can call this function back to back (cheap gas on arbitrum) to inflate their numTrained value. This numTrained value is currently only used for stats on the frontend. But if it is used for more purposes such as achievements, badges etc on the frontend, it could be a problem. 
```solidity
File: FighterFarm.sol
289:     function updateModel(
290:         uint256 tokenId, 
291:         string calldata modelHash,
292:         string calldata modelType
293:     ) 
294:         external
295:     {
296:         require(msg.sender == ownerOf(tokenId));
297:         fighters[tokenId].modelHash = modelHash; 
298:         fighters[tokenId].modelType = modelType; 
299:         numTrained[tokenId] += 1; 
300:         totalNumTrained += 1; 
301:     }
```

## [L-16] The owner of a tokenId can use up all rerolls before selling it on a marketplace

Owner can use up all rerolls for a token Id before selling it to someone on a marketplace, that would make the other buyer devoid of rerolling.
```solidity
File: FighterFarm.sol
378:     function reRoll(uint8 tokenId, uint8 fighterType) public {
379:         require(msg.sender == ownerOf(tokenId));
380:         require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
381:         require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");
```

## [L-17] Attacker can intentionally DOS user's max capacity by sending fighters from multiple accounts created

An attacker can transfer fighters (that he does not use or maybe the attacker is not involved in the protocol at all) to the user's address. This would cause the user to not be able to create more fighters, claim them or redeem them. Due to this, the user is affected.

The user can transfer these NFTs to some other address obviously but that would require the user to have that kind of skillset to interact with smart contracts directly (unless the frontend provides a feature for this transfer mechanism). 
```solidity
File: FighterFarm.sol
548:     function _ableToTransfer(uint256 tokenId, address to) private view returns(bool) {
549:         return (
550:           _isApprovedOrOwner(msg.sender, tokenId) &&
551:           balanceOf(to) < MAX_FIGHTERS_ALLOWED &&
552:           !fighterStaked[tokenId]
553:         );
554:     }
```

## [L-18] Consider spreading out probabilities for iconTypes

Since iconsType are uint8 i.e. in the range [0,255], there is a very high chance of users getting the red diamond for eyes. This is against the spec since icons are a rare sub type of champions but the eyes attribute getting red diamond most of the time makes it a common attribute. To be precise, other than 0,2,3 the remaining values in range [0,255] will always get red diamond for eyes. Since we always iterate from 0 to 5 in the for loop, it is always guaranteed to arrive at 1. 

Thus, `probability = number of favourable outcomes/total number of outcomes = 253/256` (note exlcuded 0,2,3 from favourable outcomes) = `98.82 % chance of getting red diamonds as eyes.`
```solidity
File: AiArenaHelper.sol
101:             uint256 attributesLength = attributes.length;
102:             for (uint8 i = 0; i < attributesLength; i++) { 
105:                 if (
106:                   i == 0 && iconsType == 2 || // Custom icons head (beta helmet)
107:                   i == 1 && iconsType > 0 || // Custom icons eyes (red diamond)
108:                   i == 4 && iconsType == 3 // Custom icons hands (bowling ball)
109:                 ) {
110:                     finalAttributeProbabilityIndexes[i] = 50;
```

## [L-19] Ensure sum of all probabilities of a specific attribute is always 100

First take a look at the probability indexes for each attribute in the tests [here](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/test/AiArenaHelper.t.sol#L47C4-L52C53).

```solidity
File: AiArenaHelper.t.sol
46:     function getProb() public {
47:         _probabilities.push([25, 25, 13, 13, 9, 9]); //@audit-info sum = 94
48:         _probabilities.push([25, 25, 13, 13, 9, 1]); //@audit-info sum = 86
49:         _probabilities.push([25, 25, 13, 13, 9, 10]); //@audit-info sum = 95
50:         _probabilities.push([25, 25, 13, 13, 9, 23]); //@audit-info sum = 108
51:         _probabilities.push([25, 25, 13, 13, 9, 1]); //@audit-info sum = 86
52:         _probabilities.push([25, 25, 13, 13, 9, 3]); //@audit-info sum = 84
53:     }
```

Since rarityRank is always in the range [0,99], if the sum of all attribute probabilities for a specific attribute of a generation exceeds or is less than 99 before the attributeProbabilities array length has ended (i.e. there are more elements in the array ahead), then the remaining probabilities in the array are never considered (which may include rarer probability values compared to values at previous indexes - as seen in test values).

As we can see above in the test values that the sum of all probabilities is not 100, which can cause the issue mentioned above.

```solidity
File: AiArenaHelper.sol
177:     function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute) 
178:         public 
179:         view 
180:         returns (uint256 attributeProbabilityIndex) 
181:     {
182:         uint8[] memory attrProbabilities = getAttributeProbabilities(generation, attribute);
183:         
184:         uint256 cumProb = 0; 
185:         uint256 attrProbabilitiesLength = attrProbabilities.length;
186:         for (uint8 i = 0; i < attrProbabilitiesLength; i++) {
187:             cumProb += attrProbabilities[i];
188:             if (cumProb >= rarityRank) {
189:                 attributeProbabilityIndex = i + 1;   
191:                 break;
192:             }
193:         }
194:         return attributeProbabilityIndex; 
195:     }
```


## [N-01] No need to initialize roundId to 0

The default value of an uint256 is 0, so there is no need to explicitly initialize the variable with 0.
```solidity
File: StakeAtRisk.sol
26:     /// @notice The current roundId.
27:     uint256 public roundId = 0;  
```

## [N-02] Consider using msg.sender instead of passing _ownerAddress as parameter in the constructor

_ownerAddress is supposed to be the contract deployer initially i.e. msg.sender but currently it is passed as a parameter. Although the deployer could pass his own address to fulfill this spec, it would be better to use msg.sender to set _ownerAddress in order to fully adhere to the spec as mentioned [here](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L48).
```solidity
File: Neuron.sol
73:     /// the initial supply is minted.
74:     constructor(address ownerAddress, address treasuryAddress_, address contributorAddress)
75:         ERC20("Neuron", "NRN")
76:     { 
78:         _ownerAddress = ownerAddress;
```

## [N-03] Missing event emission when transferring ownership

An event should be emitted when transferring ownership to ensure off-chain systems are notified of the critical change.
```solidity
File: Neuron.sol
91:     function transferOwnership(address newOwnerAddress) external {
92:  
93:         require(msg.sender == _ownerAddress);
94:         _ownerAddress = newOwnerAddress;
95:       
96:     }
```

## [N-04] OZ recommends using _grantRole() instead of deprecated _setupRole() function

```solidity
File: Neuron.sol
101:     function addMinter(address newMinterAddress) external {
102:         require(msg.sender == _ownerAddress);
103:         _setupRole(MINTER_ROLE, newMinterAddress);
104:     }
```
OZ AccessControl contract comment (see Line 203):
```solidity
File: AccessControl.sol
203:      * NOTE: This function is deprecated in favor of {_grantRole}.
204:      */
205:     function _setupRole(bytes32 role, address account) internal virtual {
206:         _grantRole(role, account);
207:     }
```

## [N-05] Do not decrease allowance if allowance is set to type(uint256).max

External applications/frontends usually provide users with the option to provide infinite approvals. When these infinite approvals are made, they are intentionally not decremented. This is something that is implemented by the OZ ERC20 implementation as well. Consider checking if the allowance provided by the account to the msg.sender is type(uint256).max, then do not decrease the allowance.
```solidity
File: Neuron.sol
208:     function burnFrom(address account, uint256 amount) public virtual {
209:        
210:         require(
211:             allowance(account, msg.sender) >= amount, 
212:             "ERC20: burn amount exceeds allowance"
213:         );
214:     
215:         uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
216:         _burn(account, amount);
217:         _approve(account, msg.sender, decreasedAllowance);
218:     }
```

## [N-06] Incorrect logical natspec comment should be corrected

The function adjustAdminAccess() has the natspec on Line 113 which mentions that it adjusts the admin access for a user. The word user should be replaced with trusted entity to better suit the intentions of the function.
```solidity
File: GameItems.sol
113:     /// @notice Adjusts admin access for a user.
114:     /// @dev Only the owner address is authorized to call this function.
115:     /// @param adminAddress The address of the admin.
116:     /// @param access Whether the address has admin access or not.
117:     function adjustAdminAccess(address adminAddress, bool access) external {
118:         require(msg.sender == _ownerAddress);
119:         isAdmin[adminAddress] = access;
120:     }  
```

## [N-07] Consider passing bool as value to function setAllowedBurningAddresses()

Passing bool as value would allow the admins to remove the burning addresses in case of any problem. 
```solidity
File: GameItems.sol
189:     function setAllowedBurningAddresses(address newBurningAddress) public {
190:         require(isAdmin[msg.sender]);
191:         allowedBurningAddresses[newBurningAddress] = true;
192:     }
```

## [N-08] setTokenURI() function does not check if tokenId exists

Although not directly related to the ERC1155 spec, it is always considered best practice to check if tokenId exists currently or not. This can be done using tokenId < _itemCount, which if true means the tokenId exists. 
```solidity
File: GameItems.sol
197:     function setTokenURI(uint256 tokenId, string memory _tokenURI) public {
198:         require(isAdmin[msg.sender]);
199:         _tokenURIs[tokenId] = _tokenURI;
200:     }
```

This issue also exists in the FighterFarm contract:
```solidity
File: FighterFarm.sol
186:     function setTokenURI(uint256 tokenId, string calldata newTokenURI) external {
187:         require(msg.sender == _delegatedAddress);
188:         _tokenURIs[tokenId] = newTokenURI;
189:     }
```

## [N-09] Do not hardcode string in function contractURI()

The string that stores the ipfs hash should be either passed through a setter function or made constant. 
```solidity
File: GameItems.sol
254:     function contractURI() public pure returns (string memory) {
255:         return "ipfs://bafybeih3witscmml3padf4qxbea5jh4rl2xp67aydqvqsxmyuzipwtpnii";
256:     }
```

## [N-10] In 83 years, typecast to uint32 would break in function _replenishDailyAllowance()

In 83 years, typecast to uint32 would break the protocol due to overflow of time, which would break dailyAllowanceReplenishTime 

Solution: Typecast is not required since the value in the mapping for the field being used is uint256. 
```solidity
File: GameItems.sol
318:     function _replenishDailyAllowance(uint256 tokenId) private {
319:         allowanceRemaining[msg.sender][tokenId] = allGameItemAttributes[tokenId].dailyAllowance;
320:         dailyAllowanceReplenishTime[msg.sender][tokenId] = uint32(block.timestamp + 1 days);
321:         
322:     }    
```

## [N-11] Function viewFighterInfo() does not return every data of the fighter

Some fields from the Fighter struct are excluded and not returned. 
```solidity
File: FighterOps.sol
079:     function viewFighterInfo(
080:         Fighter storage self, 
081:         address owner
082:     ) 
083:         public 
084:         view 
085:         returns (
086:             address,
087:             uint256[6] memory,
088:             uint256,
089:             uint256,
090:             string memory,
091:             string memory,
092:             uint16 
093:         )
094:     {
095:         return (
096:             owner,
097:             getFighterAttributes(self),
098:             self.weight,
099:             self.element,
100:             self.modelHash,
101:             self.modelType,
102:             self.generation
103:         );
104:     }
```

## [N-12] Reroll cost of 1000 NRN cannot be changed in the future

Missing setter would prevent the team from setting any higher or lower values to the reroll cost in the future for both types of fighterTypes. 
```solidity
File: FighterFarm.sol
42:     uint256 public rerollCost = 1000 * 10**18;    
```

## [N-13] There can only be 255 generations following which incrementGeneration() would revert

The max size for the generation array is 255. Following this, any increment would lead to an overflow error. 
```solidity
File: FighterFarm.sol
134:     function incrementGeneration(uint8 fighterType) external returns (uint8) {
135:         require(msg.sender == _ownerAddress);
136:         generation[fighterType] += 1;
137:         maxRerollsAllowed[fighterType] += 1;
138:         return generation[fighterType];
139:     }
```

## [N-14] Do not hardcode generation to 0 in constructor

Using hardcoded values is not a good practice. Consider using constants or setters that change public variables instead. 
```solidity
File: AiArenaHelper.sol
49:         addAttributeProbabilities(0, probabilities);
```