
### Low risks
Total **7 instances** over **1 risk**<br>
  

|ID|Risk|Instances|Gas Savings|
| :---: | :---: | :---: | :---: |
||L1: Some victory/combinations can stuck the game until next voltage replenish|1|0|

### Non-Critical/Quality risks
Total **63 instances** over **5 risks**<br>
  

|ID|Risk|Instances|Gas Savings|
| :---: | :---: | :---: | :---: |
|[[DOW-RVA](#DOW-RVA)]|Return values of `approve()` not checked|4|0|
|[[DOW-UCO](#DOW-UCO)]|Constants should be defined rather than using magic numbers|12|0|
|[[SWC-103](#SWC-103)]|*Lock pragmas* to specific compiler version|2|0|
|[[DOW-ICI](#DOW-ICI)]|Incorrect comparison implementation|20|0|
|[[DOW-INF](#DOW-INF)]|Large or complicated code bases should implement invariant tests|25|0|

### Gas risks
Total **146 instances** over **8 risks**<br>
  

|ID|Risk|Instances|Gas Savings|
| :---: | :---: | :---: | :---: |
|[[GAS-MEM](#GAS-MEM)]|Use *calldata* instead of *memory* for immutable arguments|37|37|
|[[GAS-UIC](#GAS-UIC)]|Use != 0 instead of > 0 for unsigned integer comparison|22|132|
|[[GAS-EIP](#GAS-EIP)]|Use *external* instead of *public* where possible|29|638|
|[[GAS-PIN](#GAS-PIN)]|Pre-increments and pre-decrements are cheaper than post-increments and post-decrements|21|105|
|[[GAS-CUE](#GAS-CUE)]|Use custom errors|19|19|
|[[GAS-SRS](#GAS-SRS)]|Splitting *require()* statements that use *&&* saves gas|1|3|
|[[GAS-IUL](#GAS-IUL)]|Increments can be unchecked in for-loops|13|455|
|[[GAS-12](#GAS-12)]|Use *abi.encodePacked()* instead of *abi.encode()*|4|3588|

## Details contents

## Low risks

# L1: Some victory/combinations can stuck the game until next voltage replenish #

# Lines of code

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L235


# Vulnerability details

## Impact
At the expected end of the round, if `totalAccumulatedPoints` is equal to 0, it will be impossible to start a new round. The game will be stuck until some player can play again with voltage replenish.

## Proof of Concept

The easiest poc is when only two players only tie during a round.
The following test can show this case:
https://github.com/desaperh/AI_ARENA/blob/main/RankedBattle_dowsers.t.sol

More complex cases can be added. The primary challenge lies in modeling Elo modifications after a match.

## Tools Used
Foundry Invariant Testing

## Recommended Mitigation Steps
Without modifying the code, administrators must ensure there is a fighter owned by administrators ready to participate in a match at the end of the round. Conducting some matches will effectively resolve the situation.
Alternative way : 
https://github.com/desaperh/AI_ARENA/blob/main/RankedBattle_modified.sol
The value 1 can be set to a more suitable value for rewards calculation


## Assessed type

Other
## Non-Critical/Quality risks

### [DOW-RVA]<a name="DOW-RVA"></a> Return values of `approve()` not checked
**Description:**<br>
Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything<br>
https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca<br>**Recommendation:**<br>
It might be useful to check the return value, even if it is gas costs. if the approval fails, the transaction would revert to initial state. <br>
<br>
*There are 4 instances of this risk:*
<details><summary>see instances</summary>File: Neuron.sol


```solidity
131: 
            _approve(
175: 
        _approve(
188: 
        _approve(
202: 
        _approve(

```
*GitHub* : [L131-L132](/Neuron.sol#L131-L132) [L175-L176](/Neuron.sol#L175-L176) [L188-L189](/Neuron.sol#L188-L189) [L202-L203](/Neuron.sol#L202-L203) 

<br>
</details>

### [DOW-UCO]<a name="DOW-UCO"></a> Constants should be defined rather than using magic numbers
**Description:**<br>
Constants should be defined rather than using magic numbers<br>
<br>**Recommendation:**<br>
cf. Description<br>
<br>
*There are 12 instances of this risk:*
<details><summary>see instances</summary>File: AiArenaHelper.sol


```solidity
20: ublic defaultAttributeDivisor = [2, 3, 5, 7, 11, 13
94:             return FighterOps.FighterPhysicalAttributes(99, 99, 99, 99, 99, 99
105:                     finalAttributeProbabilityIndexes[i] = 50

```
*GitHub* : [L20](/AiArenaHelper.sol#L20) [L94](/AiArenaHelper.sol#L94) [L105](/AiArenaHelper.sol#L105) 

<br>File: FighterFarm.sol


```solidity
471: nt256 weight = dna % 31 + 65

```
*GitHub* : [L471](/FighterFarm.sol#L471) 

<br>File: RankedBattle.sol


```solidity
157:         rankedNrnDistribution[0] = 5000

```
*GitHub* : [L157](/RankedBattle.sol#L157) 

<br>File: FixedPointMathLib.sol


```solidity
177:                 z := shl(64, z) // Like multiplying by 2 ** 64
180:                 y := shr(64, y) // Like dividing by 2 ** 64
185:                 z := shl(16, z) // Like multiplying by 2 ** 16
188:                 y := shr(16, y) // Like dividing by 2 ** 16

```
*GitHub* : [L177](/FixedPointMathLib.sol#L177) [L180](/FixedPointMathLib.sol#L180) [L185](/FixedPointMathLib.sol#L185) [L188](/FixedPointMathLib.sol#L188) 

<br>File: Verification.sol


```solidity
15:         require(signature.length == 65
34:             s := mload(add(signature, 64
36:             v := byte(0, mload(add(signature, 96

```
*GitHub* : [L15](/Verification.sol#L15) [L34](/Verification.sol#L34) [L36](/Verification.sol#L36) 

<br>
</details>

### [SWC-103]<a name="SWC-103"></a> *Lock pragmas* to specific compiler version
**Description:**<br>Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.<br>https://swcregistry.io/docs/SWC-103 <br>**Recommendation:**<br>Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.<br>https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/<br>
*There are 2 instances of this risk:*
<details><summary>see instances</summary>File: AiArenaHelper.sol


```solidity
2: pragma solidity ^0.8.20

```
*GitHub* : [L2](/AiArenaHelper.sol#L2) 

<br>File: AAMintPass.sol


```solidity
2: pragma solidity ^0.8.13

```
*GitHub* : [L2](/AAMintPass.sol#L2) 

<br>
</details>

### [DOW-ICI]<a name="DOW-ICI"></a> Incorrect comparison implementation
**Description:**<br>
Comparison will be ignored. and does not have any edge cases with unexpected behavior. <br>**Recommendation:**<br>
Using *require* or *if* statements for comparisons can help prevent logical errors<br>
<br>
*There are 20 instances of this risk:*
<details><summary>see instances</summary>File: MergingPool.sol


```solidity
153: (msg.sender == winnerAddresses[currentRound][j])
179: (claimer == winnerAddresses[currentRound][j])

```
*GitHub* : [L153](/MergingPool.sol#L153) [L179](/MergingPool.sol#L179) 

<br>File: FighterFarm.sol


```solidity
499: (customAttributes[0] == 100)

```
*GitHub* : [L499](/FighterFarm.sol#L499) 

<br>File: RankedBattle.sol


```solidity
246: (tokenId) == msg.sender, "Caller does not own fighter")
253: (amountStaked[tokenId] == 0)
271: (tokenId) == msg.sender, "Caller does not own fighter")
285: (amountStaked[tokenId] == 0)
433: (_calculatedStakingFactor[tokenId][roundId] == false)
440: (battleResult == 0)
444: (stakeAtRisk == 0)
472: (battleResult == 2)
506: (battleResult == 0)
508: (battleResult == 1)
510: (battleResult == 2)
530: (stakingFactor_ == 0)

```
*GitHub* : [L246](/RankedBattle.sol#L246) [L253](/RankedBattle.sol#L253) [L271](/RankedBattle.sol#L271) [L285](/RankedBattle.sol#L285) [L433](/RankedBattle.sol#L433) [L440](/RankedBattle.sol#L440) [L444](/RankedBattle.sol#L444) [L472](/RankedBattle.sol#L472) [L506](/RankedBattle.sol#L506) [L508](/RankedBattle.sol#L508) [L510](/RankedBattle.sol#L510) [L530](/RankedBattle.sol#L530) 

<br>File: GameItems.sol


```solidity
166: (dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp)
270: (dailyAllowanceReplenishTime[owner][tokenId] <= block.timestamp)

```
*GitHub* : [L166](/GameItems.sol#L166) [L270](/GameItems.sol#L270) 

<br>File: FixedPointMathLib.sol


```solidity
43: (x == 0 || (x * y) / x == y))
62: (x == 0 || (x * y) / x == y))

```
*GitHub* : [L43](/FixedPointMathLib.sol#L43) [L62](/FixedPointMathLib.sol#L62) 

<br>File: VoltageManager.sol


```solidity
107: (ownerVoltageReplenishTime[spender] <= block.timestamp)

```
*GitHub* : [L107](/VoltageManager.sol#L107) 

<br>
</details>

### [DOW-INF]<a name="DOW-INF"></a> Large or complicated code bases should implement invariant tests
**Description:**<br>
This includes: large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts. Invariant fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100'%' code coverage can still have bugs due to the order of the operations a user performs, and invariant fuzzers may help significantly.<br>
<br>**Recommendation:**<br>
Provide the test base to ensure the invariant holds<br>
<br>
*There are 25 instances of this risk:*
<details><summary>see instances</summary>File: MergingPool.sol


```solidity
169: spec
202: spec

```
*GitHub* : [L169](/MergingPool.sol#L169) [L202](/MergingPool.sol#L202) 

<br>File: FighterFarm.sol


```solidity
125: spec
187: spec
419: spec
476: spec
533: spec

```
*GitHub* : [L125](/FighterFarm.sol#L125) [L187](/FighterFarm.sol#L187) [L419](/FighterFarm.sol#L419) [L476](/FighterFarm.sol#L476) [L533](/FighterFarm.sol#L533) 

<br>File: StakeAtRisk.sol


```solidity
129: spec

```
*GitHub* : [L129](/StakeAtRisk.sol#L129) 

<br>File: RankedBattle.sol


```solidity
292: spec
378: spec
383: spec

```
*GitHub* : [L292](/RankedBattle.sol#L292) [L378](/RankedBattle.sol#L378) [L383](/RankedBattle.sol#L383) 

<br>File: GameItems.sol


```solidity
124: spec
192: spec
199: spec
237: spec
254: spec
266: spec
276: spec
309: spec

```
*GitHub* : [L124](/GameItems.sol#L124) [L192](/GameItems.sol#L192) [L199](/GameItems.sol#L199) [L237](/GameItems.sol#L237) [L254](/GameItems.sol#L254) [L266](/GameItems.sol#L266) [L276](/GameItems.sol#L276) [L309](/GameItems.sol#L309) 

<br>File: Neuron.sol


```solidity
136: spec
151: spec
161: spec
167: spec
179: spec
192: spec

```
*GitHub* : [L136](/Neuron.sol#L136) [L151](/Neuron.sol#L151) [L161](/Neuron.sol#L161) [L167](/Neuron.sol#L167) [L179](/Neuron.sol#L179) [L192](/Neuron.sol#L192) 

<br>
</details>

## Gas risks

### [GAS-MEM]<a name="GAS-MEM"></a> Use *calldata* instead of *memory* for immutable arguments
**Description:**<br>
If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as *calldata*. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies *memory* storage.<br>
<br><br>**Recommendation:**<br>
Mark data types as *calldata* instead of *memory* where possible. This makes it so that the data is not automatically loaded into memory.<br>
*There are 37 instances of this risk:*
<details><summary>see instances</summary>File: AiArenaHelper.sol


```solidity
41:     constructor(uint8[][] memory probabilities) {
68:     function addAttributeDivisor(uint8[] memory attributeDivisors) external {
91:         returns (FighterOps.FighterPhysicalAttributes memory) 
96:             uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributes.length);
131:     function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
157:     function getAttributeProbabilities(uint256 generation, string memory attribute) 
160:         returns (uint8[] memory) 
169:     function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute) 
174:         uint8[] memory attrProbabilities = getAttributeProbabilities(generation, attribute);

```
*GitHub* : [L41](/AiArenaHelper.sol#L41) [L68](/AiArenaHelper.sol#L68) [L91](/AiArenaHelper.sol#L91) [L96](/AiArenaHelper.sol#L96) [L131](/AiArenaHelper.sol#L131) [L157](/AiArenaHelper.sol#L157) [L160](/AiArenaHelper.sol#L160) [L169](/AiArenaHelper.sol#L169) [L174](/AiArenaHelper.sol#L174) 

<br>File: MergingPool.sol


```solidity
123:         address[] memory currentWinnerAddresses = new address[](winnersLength);
205:     function getFighterPoints(uint256 maxId) public view returns(uint256[] memory) {
206:         uint256[] memory points = new uint256[](1);

```
*GitHub* : [L123](/MergingPool.sol#L123) [L205](/MergingPool.sol#L205) [L206](/MergingPool.sol#L206) 

<br>File: FighterFarm.sol


```solidity
395:     function contractURI() public pure returns (string memory) {
402:     function tokenURI(uint256 tokenId) public view override(ERC721) returns (string memory) {
428:             uint256[6] memory,
431:             string memory,
432:             string memory,
487:         string memory modelHash,
488:         string memory modelType, 
491:         uint256[2] memory customAttributes
510:         FighterOps.FighterPhysicalAttributes memory attrs = _aiArenaHelperInstance.createPhysicalAttributes(

```
*GitHub* : [L395](/FighterFarm.sol#L395) [L402](/FighterFarm.sol#L402) [L428](/FighterFarm.sol#L428) [L431](/FighterFarm.sol#L431) [L432](/FighterFarm.sol#L432) [L487](/FighterFarm.sol#L487) [L488](/FighterFarm.sol#L488) [L491](/FighterFarm.sol#L491) [L510](/FighterFarm.sol#L510) 

<br>File: GameItems.sol


```solidity
194:     function setTokenURI(uint256 tokenId, string memory _tokenURI) public {
209:         string memory name_,
210:         string memory tokenURI,
249:     function contractURI() public pure returns (string memory) {
256:     function uri(uint256 tokenId) public view override returns (string memory) {
257:         string memory customURI = _tokenURIs[tokenId];
296:         bytes memory data

```
*GitHub* : [L194](/GameItems.sol#L194) [L209](/GameItems.sol#L209) [L210](/GameItems.sol#L210) [L249](/GameItems.sol#L249) [L256](/GameItems.sol#L256) [L257](/GameItems.sol#L257) [L296](/GameItems.sol#L296) 

<br>File: FighterOps.sol


```solidity
67:     function getFighterAttributes(Fighter storage self) public view returns (uint256[6] memory) {
87:             uint256[6] memory,
90:             string memory,
91:             string memory,

```
*GitHub* : [L67](/FighterOps.sol#L67) [L87](/FighterOps.sol#L87) [L90](/FighterOps.sol#L90) [L91](/FighterOps.sol#L91) 

<br>File: AAMintPass.sol


```solidity
150:     function contractURI() public pure returns (string memory) {
156:     function tokenURI(uint256 _tokenId) public view override(ERC721) returns (string memory) {

```
*GitHub* : [L150](/AAMintPass.sol#L150) [L156](/AAMintPass.sol#L156) 

<br>File: Verification.sol


```solidity
8:         bytes memory signature,
28:             mload(p) loads next 32 bytes starting at the memory address p into memory
38:         bytes memory prefix = "\x19Ethereum Signed Message:\n32";

```
*GitHub* : [L8](/Verification.sol#L8) [L28](/Verification.sol#L28) [L38](/Verification.sol#L38) 

<br>
</details>

### [GAS-UIC]<a name="GAS-UIC"></a> Use != 0 instead of > 0 for unsigned integer comparison
**Description:**<br>
*!= 0* costs **less gas than > 0** <br>
See for more information:<br>
[Source1](https://gist.github.com/ahmedovv123/ac550b389bcbe21043c613e6c6c1b563#GAS-9) <br>
[Source2](https://gist.github.com/IllIllI000/bf2c3120f24a69e489f12b3213c06c94) <br>
<br>**Recommendation:**<br>
Use *!= 0* instead of *> 0* for unsigned integer comparison in:<br>
<br>
*There are 22 instances of this risk:*
<details><summary>see instances</summary>File: AiArenaHelper.sol


```solidity
3: 0 <0
102: > 0

```
*GitHub* : [L3](/AiArenaHelper.sol#L3) [L102](/AiArenaHelper.sol#L102) 

<br>File: MergingPool.sol


```solidity
2: 0 <0
164: > 0

```
*GitHub* : [L2](/MergingPool.sol#L2) [L164](/MergingPool.sol#L164) 

<br>File: FighterFarm.sol


```solidity
2: 0 <0

```
*GitHub* : [L2](/FighterFarm.sol#L2) 

<br>File: StakeAtRisk.sol


```solidity
2: 0 <0

```
*GitHub* : [L2](/StakeAtRisk.sol#L2) 

<br>File: RankedBattle.sol


```solidity
2: 0 <0
235: > 0
245: > 0
306: > 0
342: > 0
460: > 0
469: > 0
479: > 0
488: > 0

```
*GitHub* : [L2](/RankedBattle.sol#L2) [L235](/RankedBattle.sol#L235) [L245](/RankedBattle.sol#L245) [L306](/RankedBattle.sol#L306) [L342](/RankedBattle.sol#L342) [L460](/RankedBattle.sol#L460) [L469](/RankedBattle.sol#L469) [L479](/RankedBattle.sol#L479) [L488](/RankedBattle.sol#L488) 

<br>File: GameItems.sol


```solidity
2: 0 <0
258: > 0

```
*GitHub* : [L2](/GameItems.sol#L2) [L258](/GameItems.sol#L258) 

<br>File: VoltageManager.sol


```solidity
2: 0 <0
95: > 0

```
*GitHub* : [L2](/VoltageManager.sol#L2) [L95](/VoltageManager.sol#L95) 

<br>File: FighterOps.sol


```solidity
2: 0 <0

```
*GitHub* : [L2](/FighterOps.sol#L2) 

<br>File: Neuron.sol


```solidity
2: 0 <0

```
*GitHub* : [L2](/Neuron.sol#L2) 

<br>File: Verification.sol


```solidity
2: 0 <0

```
*GitHub* : [L2](/Verification.sol#L2) 

<br>
</details>

### [GAS-EIP]<a name="GAS-EIP"></a> Use *external* instead of *public* where possible
**Description:**<br>
Functions with the public visibility modifier are costlier than external. <br>
<br>**Recommendation:**<br>
Default to using the external modifier until you need to expose it to other functions within your contract.See [source](https://yos.io/2021/05/17/gas-efficient-solidity/#tip-17-use-external-instead-of-public-where-possible).<br>
<br>
*There are 29 instances of this risk:*
<details><summary>see instances</summary>File: AiArenaHelper.sol


```solidity
131: function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public
144: function deleteAttributeProbabilities(uint8 generation) public

```
*GitHub* : [L131](/AiArenaHelper.sol#L131) [L144](/AiArenaHelper.sol#L144) 

<br>File: MergingPool.sol


```solidity
195: function addPoints(uint256 tokenId, uint256 points) public
205: function getFighterPoints(uint256 maxId) public

```
*GitHub* : [L195](/MergingPool.sol#L195) [L205](/MergingPool.sol#L205) 

<br>File: FighterFarm.sol


```solidity
370: function reRoll(uint8 tokenId, uint8 fighterType) public
395: function contractURI() public
402: function tokenURI(uint256 tokenId) public

```
*GitHub* : [L370](/FighterFarm.sol#L370) [L395](/FighterFarm.sol#L395) [L402](/FighterFarm.sol#L402) 

<br>File: RankedBattle.sol


```solidity
379: function getNrnDistribution(uint256 roundId_) public
386: function getUnclaimedNRN(address claimer) public

```
*GitHub* : [L379](/RankedBattle.sol#L379) [L386](/RankedBattle.sol#L386) 

<br>File: GameItems.sol


```solidity
185: function setAllowedBurningAddresses(address newBurningAddress) public
194: function setTokenURI(uint256 tokenId, string memory _tokenURI) public
242: function burn(address account, uint256 tokenId, uint256 amount) public
249: function contractURI() public
256: function uri(uint256 tokenId) public
268: function getAllowanceRemaining(address owner, uint256 tokenId) public
279: function remainingSupply(uint256 tokenId) public
285: function uniqueTokensOutstanding() public

```
*GitHub* : [L185](/GameItems.sol#L185) [L194](/GameItems.sol#L194) [L242](/GameItems.sol#L242) [L249](/GameItems.sol#L249) [L256](/GameItems.sol#L256) [L268](/GameItems.sol#L268) [L279](/GameItems.sol#L279) [L285](/GameItems.sol#L285) 

<br>File: VoltageManager.sol


```solidity
93: function useVoltageBattery() public
105: function spendVoltage(address spender, uint8 voltageSpent) public

```
*GitHub* : [L93](/VoltageManager.sol#L93) [L105](/VoltageManager.sol#L105) 

<br>File: FighterOps.sol


```solidity
67: function getFighterAttributes(Fighter storage self) public

```
*GitHub* : [L67](/FighterOps.sol#L67) 

<br>File: Neuron.sol


```solidity
155: function mint(address to, uint256 amount) public
163: function burn(uint256 amount) public
171: function approveSpender(address account, uint256 amount) public
184: function approveStaker(address owner, address spender, uint256 amount) public
196: function burnFrom(address account, uint256 amount) public

```
*GitHub* : [L155](/Neuron.sol#L155) [L163](/Neuron.sol#L163) [L171](/Neuron.sol#L171) [L184](/Neuron.sol#L184) [L196](/Neuron.sol#L196) 

<br>File: AAMintPass.sol


```solidity
137: function burn(uint256 _tokenId) public
145: function totalSupply() public
150: function contractURI() public
156: function tokenURI(uint256 _tokenId) public

```
*GitHub* : [L137](/AAMintPass.sol#L137) [L145](/AAMintPass.sol#L145) [L150](/AAMintPass.sol#L150) [L156](/AAMintPass.sol#L156) 

<br>
</details>

### [GAS-PIN]<a name="GAS-PIN"></a> Pre-increments and pre-decrements are cheaper than post-increments and post-decrements
**Description:**<br>
*++i* costs **less gas than i++**, especially when it's used in loops (i-- too).
See for more information:
https://gist.github.com/ahmedovv123/ac550b389bcbe21043c613e6c6c1b563#GAS-5 <br>
<br>**Recommendation:**<br>
Use *++i* or *--i* in: <br>
<br>
*There are 21 instances of this risk:*
<details><summary>see instances</summary>File: AiArenaHelper.sol


```solidity
48: i++
73: i++
99: i++
136: i++
148: i++
178: i++

```
*GitHub* : [L48](/AiArenaHelper.sol#L48) [L73](/AiArenaHelper.sol#L73) [L99](/AiArenaHelper.sol#L99) [L136](/AiArenaHelper.sol#L136) [L148](/AiArenaHelper.sol#L148) [L178](/AiArenaHelper.sol#L178) 

<br>File: MergingPool.sol


```solidity
124: i++
149: currentRound++
152: j++
176: currentRound++
178: j++
207: i++

```
*GitHub* : [L124](/MergingPool.sol#L124) [L149](/MergingPool.sol#L149) [L152](/MergingPool.sol#L152) [L176](/MergingPool.sol#L176) [L178](/MergingPool.sol#L178) [L207](/MergingPool.sol#L207) 

<br>File: FighterFarm.sol


```solidity
211: i++
249: i++

```
*GitHub* : [L211](/FighterFarm.sol#L211) [L249](/FighterFarm.sol#L249) 

<br>File: RankedBattle.sol


```solidity
299: currentRound++
390: i++

```
*GitHub* : [L299](/RankedBattle.sol#L299) [L390](/RankedBattle.sol#L390) 

<br>File: Neuron.sol


```solidity
131: i++

```
*GitHub* : [L131](/Neuron.sol#L131) 

<br>File: AAMintPass.sol


```solidity
130: i++
139: numTokensBurned++
140: numTokensOutstanding--
165: numTokensOutstanding++

```
*GitHub* : [L130](/AAMintPass.sol#L130) [L139](/AAMintPass.sol#L139) [L140](/AAMintPass.sol#L140) [L165](/AAMintPass.sol#L165) 

<br>
</details>

### [GAS-CUE]<a name="GAS-CUE"></a> Use custom errors
**Description:**<br>
Using error strings is gaz expensive.<br>
<br>**Recommendation:**<br>
Use Custom Errors. This would save both deployment and runtime cost.[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
<br>
<br>
*There are 19 instances of this risk:*
<details><summary>see instances</summary>File: AiArenaHelper.sol


```solidity
133: require(probabilities.length == 6, "Invalid number of attribute arrays")

```
*GitHub* : [L133](/AiArenaHelper.sol#L133) 

<br>File: MergingPool.sol


```solidity
120: require(winners.length == winnersPerPeriod, "Incorrect number of winners")
121: require(!isSelectionComplete[roundId], "Winners are already selected")
196: require(msg.sender == _rankedBattleAddress, "Not Ranked Battle contract address")

```
*GitHub* : [L120](/MergingPool.sol#L120) [L121](/MergingPool.sol#L121) [L196](/MergingPool.sol#L196) 

<br>File: FighterFarm.sol


```solidity
373: require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll")

```
*GitHub* : [L373](/FighterFarm.sol#L373) 

<br>File: StakeAtRisk.sol


```solidity
79: require(msg.sender == _rankedBattleAddress, "Not authorized to set new round")
94: require(msg.sender == _rankedBattleAddress, "Call must be from RankedBattle contract")
122: require(msg.sender == _rankedBattleAddress, "Call must be from RankedBattle contract")

```
*GitHub* : [L79](/StakeAtRisk.sol#L79) [L94](/StakeAtRisk.sol#L94) [L122](/StakeAtRisk.sol#L122) 

<br>File: RankedBattle.sol


```solidity
245: require(amount > 0, "Amount cannot be 0")
246: require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter")
247: require(_neuronInstance.balanceOf(msg.sender) >= amount, "Stake amount exceeds balance")
248: require(hasUnstaked[tokenId][roundId] == false, "Cannot add stake after unstaking this round")
271: require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter")
295: require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period")

```
*GitHub* : [L245](/RankedBattle.sol#L245) [L246](/RankedBattle.sol#L246) [L247](/RankedBattle.sol#L247) [L248](/RankedBattle.sol#L248) [L271](/RankedBattle.sol#L271) [L295](/RankedBattle.sol#L295) 

<br>File: GameItems.sol


```solidity
150: require(_neuronInstance.balanceOf(msg.sender) >= price, "Not enough NRN for purchase")

```
*GitHub* : [L150](/GameItems.sol#L150) 

<br>File: Neuron.sol


```solidity
156: require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply")
157: require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint")

```
*GitHub* : [L156](/Neuron.sol#L156) [L157](/Neuron.sol#L157) 

<br>File: AAMintPass.sol


```solidity
157: require(_exists(_tokenId), "ERC721Metadata: URI query for nonexistent token")

```
*GitHub* : [L157](/AAMintPass.sol#L157) 

<br>File: Verification.sol


```solidity
15: require(signature.length == 65, "invalid signature length")

```
*GitHub* : [L15](/Verification.sol#L15) 

<br>
</details>

### [GAS-SRS]<a name="GAS-SRS"></a> Splitting *require()* statements that use *&&* saves gas
**Description:**<br>
Using the *&&* operator in a single *require* statement to check multiple conditions cost more gas than multiple *require* statements with one condition.See for more information:
https://yos.io/2021/05/17/gas-efficient-solidity/#tip-11-splitting-require-statements-that-use--saves-gas <br>
<br>**Recommendation:**<br>
Use multiple require statements with one condition<br>
<br>
*There is 1 instance of this risk:*
<details><summary>see instance</summary>File: FighterFarm.sol


```solidity
208: require(modelHashes.length == totalToMint && modelTypes.length == totalToMint);

```
*GitHub* : [L208](/FighterFarm.sol#L208) 

<br>
</details>

### [GAS-IUL]<a name="GAS-IUL"></a> Increments can be unchecked in for-loops
**Description:**<br>
Use ++i or --i in **solidity version 0.8.0**, should be unchecked{++i}/unchecked{--i} when it is not possible for them to overflow, as is the case when used in FOR and WHILE loops The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP
<br>
<br>**Recommendation:**<br>
Use *unchecked{++i/i++}* or *unchecked{--i/i--}*<br>
<br>
<br>
*There are 13 instances of this risk:*
<details><summary>see instances</summary>File: AiArenaHelper.sol


```solidity
48: for (uint8 i = 0; i < attributesLength; i++)
73: for (uint8 i = 0; i < attributesLength; i++)
99: for (uint8 i = 0; i < attributesLength; i++)
136: for (uint8 i = 0; i < attributesLength; i++)
148: for (uint8 i = 0; i < attributesLength; i++)
178: for (uint8 i = 0; i < attrProbabilitiesLength; i++)

```
*GitHub* : [L48](/AiArenaHelper.sol#L48) [L73](/AiArenaHelper.sol#L73) [L99](/AiArenaHelper.sol#L99) [L136](/AiArenaHelper.sol#L136) [L148](/AiArenaHelper.sol#L148) [L178](/AiArenaHelper.sol#L178) 

<br>File: MergingPool.sol


```solidity
124: for (uint256 i = 0; i < winnersLength; i++)
207: for (uint256 i = 0; i < maxId; i++)

```
*GitHub* : [L124](/MergingPool.sol#L124) [L207](/MergingPool.sol#L207) 

<br>File: FighterFarm.sol


```solidity
211: for (uint16 i = 0; i < totalToMint; i++)
249: for (uint16 i = 0; i < mintpassIdsToBurn.length; i++)

```
*GitHub* : [L211](/FighterFarm.sol#L211) [L249](/FighterFarm.sol#L249) 

<br>File: RankedBattle.sol


```solidity
390: for (uint32 i = lowerBound; i < roundId; i++)

```
*GitHub* : [L390](/RankedBattle.sol#L390) 

<br>File: Neuron.sol


```solidity
131: for (uint32 i = 0; i < recipientsLength; i++)

```
*GitHub* : [L131](/Neuron.sol#L131) 

<br>File: AAMintPass.sol


```solidity
130: for (uint16 i = 0; i < totalToMint; i++)

```
*GitHub* : [L130](/AAMintPass.sol#L130) 

<br>
</details>

### [GAS-12]<a name="GAS-12"></a> Use *abi.encodePacked()* instead of *abi.encode()*
**Description:**<br>abi.encode() is less efficient than abi.encodePacked().<br>See for more information:<br>https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison<br>**Recommendation:**<br>
Use *abi.encodePacked()* on Uint and String type instead<br>
*There are 4 instances of this risk:*
<details><summary>see instances</summary>File: FighterFarm.sol


```solidity
214: abi.encode(msg.sender, fighters.length)))
254: abi.encode(mintPassDnas[i])))
324: abi.encode(msg.sender, fighters.length)))
379: abi.encode(msg.sender, tokenId, numRerolls[tokenId])))

```
*GitHub* : [L214](/FighterFarm.sol#L214) [L254](/FighterFarm.sol#L254) [L324](/FighterFarm.sol#L324) [L379](/FighterFarm.sol#L379) 

<br>
</details>