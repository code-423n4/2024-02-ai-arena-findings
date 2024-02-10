# c4udit Report

## Files analyzed
- 2024-02-ai-arena/src/AiArenaHelper.sol
- 2024-02-ai-arena/src/FighterFarm.sol
- 2024-02-ai-arena/src/GameItems.sol
- 2024-02-ai-arena/src/MergingPool.sol
- 2024-02-ai-arena/src/Neuron.sol
- 2024-02-ai-arena/src/RankedBattle.sol
- 2024-02-ai-arena/src/StakeAtRisk.sol
- 2024-02-ai-arena/src/VoltageManager.sol

## Issues found

### Don't Initialize Variables with Default Value

#### Impact
Issue Information: [G001](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g001---dont-initialize-variables-with-default-value)

#### Findings:
```
2024-02-ai-arena/src/AiArenaHelper.sol::48 => for (uint8 i = 0; i < attributesLength; i++) {
2024-02-ai-arena/src/AiArenaHelper.sol::73 => for (uint8 i = 0; i < attributesLength; i++) {
2024-02-ai-arena/src/AiArenaHelper.sol::99 => for (uint8 i = 0; i < attributesLength; i++) {
2024-02-ai-arena/src/AiArenaHelper.sol::136 => for (uint8 i = 0; i < attributesLength; i++) {
2024-02-ai-arena/src/AiArenaHelper.sol::148 => for (uint8 i = 0; i < attributesLength; i++) {
2024-02-ai-arena/src/AiArenaHelper.sol::176 => uint256 cumProb = 0;
2024-02-ai-arena/src/AiArenaHelper.sol::178 => for (uint8 i = 0; i < attrProbabilitiesLength; i++) {
2024-02-ai-arena/src/FighterFarm.sol::211 => for (uint16 i = 0; i < totalToMint; i++) {
2024-02-ai-arena/src/FighterFarm.sol::249 => for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
2024-02-ai-arena/src/MergingPool.sol::124 => for (uint256 i = 0; i < winnersLength; i++) {
2024-02-ai-arena/src/MergingPool.sol::147 => uint32 claimIndex = 0;
2024-02-ai-arena/src/MergingPool.sol::152 => for (uint32 j = 0; j < winnersLength; j++) {
2024-02-ai-arena/src/MergingPool.sol::174 => uint256 numRewards = 0;
2024-02-ai-arena/src/MergingPool.sol::178 => for (uint32 j = 0; j < winnersLength; j++) {
2024-02-ai-arena/src/MergingPool.sol::207 => for (uint256 i = 0; i < maxId; i++) {
2024-02-ai-arena/src/Neuron.sol::131 => for (uint32 i = 0; i < recipientsLength; i++) {
2024-02-ai-arena/src/RankedBattle.sol::296 => uint256 claimableNRN = 0;
2024-02-ai-arena/src/RankedBattle.sol::387 => uint256 claimableNRN = 0;
2024-02-ai-arena/src/RankedBattle.sol::427 => uint256 points = 0;
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Cache Array Length Outside of Loop

#### Impact
Issue Information: [G002](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g002---cache-array-length-outside-of-loop)

#### Findings:
```
2024-02-ai-arena/src/AiArenaHelper.sol::47 => uint256 attributesLength = attributes.length;
2024-02-ai-arena/src/AiArenaHelper.sol::70 => require(attributeDivisors.length == attributes.length);
2024-02-ai-arena/src/AiArenaHelper.sol::72 => uint256 attributesLength = attributes.length;
2024-02-ai-arena/src/AiArenaHelper.sol::96 => uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributes.length);
2024-02-ai-arena/src/AiArenaHelper.sol::98 => uint256 attributesLength = attributes.length;
2024-02-ai-arena/src/AiArenaHelper.sol::133 => require(probabilities.length == 6, "Invalid number of attribute arrays");
2024-02-ai-arena/src/AiArenaHelper.sol::135 => uint256 attributesLength = attributes.length;
2024-02-ai-arena/src/AiArenaHelper.sol::147 => uint256 attributesLength = attributes.length;
2024-02-ai-arena/src/AiArenaHelper.sol::177 => uint256 attrProbabilitiesLength = attrProbabilities.length;
2024-02-ai-arena/src/FighterFarm.sol::208 => require(modelHashes.length == totalToMint && modelTypes.length == totalToMint);
2024-02-ai-arena/src/FighterFarm.sol::214 => uint256(keccak256(abi.encode(msg.sender, fighters.length))),
2024-02-ai-arena/src/FighterFarm.sol::225 => /// @dev This function requires the length of all input arrays to be equal.
2024-02-ai-arena/src/FighterFarm.sol::244 => mintpassIdsToBurn.length == mintPassDnas.length &&
2024-02-ai-arena/src/FighterFarm.sol::245 => mintPassDnas.length == fighterTypes.length &&
2024-02-ai-arena/src/FighterFarm.sol::246 => fighterTypes.length == modelHashes.length &&
2024-02-ai-arena/src/FighterFarm.sol::247 => modelHashes.length == modelTypes.length
2024-02-ai-arena/src/FighterFarm.sol::249 => for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
2024-02-ai-arena/src/FighterFarm.sol::324 => uint256(keccak256(abi.encode(msg.sender, fighters.length))),
2024-02-ai-arena/src/FighterFarm.sol::507 => uint256 newId = fighters.length;
2024-02-ai-arena/src/GameItems.sol::258 => if (bytes(customURI).length > 0) {
2024-02-ai-arena/src/GameItems.sol::286 => return allGameItemAttributes.length;
2024-02-ai-arena/src/MergingPool.sol::120 => require(winners.length == winnersPerPeriod, "Incorrect number of winners");
2024-02-ai-arena/src/MergingPool.sol::122 => uint256 winnersLength = winners.length;
2024-02-ai-arena/src/MergingPool.sol::151 => winnersLength = winnerAddresses[currentRound].length;
2024-02-ai-arena/src/MergingPool.sol::177 => winnersLength = winnerAddresses[currentRound].length;
2024-02-ai-arena/src/Neuron.sol::129 => require(recipients.length == amounts.length);
2024-02-ai-arena/src/Neuron.sol::130 => uint256 recipientsLength = recipients.length;
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Use != 0 instead of > 0 for Unsigned Integer Comparison

#### Impact
Issue Information: [G003](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g003---use--0-instead-of--0-for-unsigned-integer-comparison)

#### Findings:
```
2024-02-ai-arena/src/AiArenaHelper.sol::102 => i == 1 && iconsType > 0 || // Custom icons eyes (red diamond)
2024-02-ai-arena/src/GameItems.sol::258 => if (bytes(customURI).length > 0) {
2024-02-ai-arena/src/MergingPool.sol::164 => if (claimIndex > 0) {
2024-02-ai-arena/src/RankedBattle.sol::235 => require(totalAccumulatedPoints[roundId] > 0);
2024-02-ai-arena/src/RankedBattle.sol::245 => require(amount > 0, "Amount cannot be 0");
2024-02-ai-arena/src/RankedBattle.sol::306 => if (claimableNRN > 0) {
2024-02-ai-arena/src/RankedBattle.sol::342 => if (amountStaked[tokenId] + stakeAtRisk > 0) {
2024-02-ai-arena/src/RankedBattle.sol::460 => if (curStakeAtRisk > 0) {
2024-02-ai-arena/src/RankedBattle.sol::469 => if (points > 0) {
2024-02-ai-arena/src/RankedBattle.sol::479 => if (accumulatedPointsPerFighter[tokenId][roundId] > 0) {
2024-02-ai-arena/src/RankedBattle.sol::488 => if (points > 0) {
2024-02-ai-arena/src/VoltageManager.sol::95 => require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Use immutable for OpenZeppelin AccessControl's Roles Declarations

#### Impact
Issue Information: [G006](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g006---use-immutable-for-openzeppelin-accesscontrols-roles-declarations)

#### Findings:
```
2024-02-ai-arena/src/FighterFarm.sol::199 => bytes32 msgHash = bytes32(keccak256(abi.encode(
2024-02-ai-arena/src/FighterFarm.sol::214 => uint256(keccak256(abi.encode(msg.sender, fighters.length))),
2024-02-ai-arena/src/FighterFarm.sol::254 => uint256(keccak256(abi.encode(mintPassDnas[i]))),
2024-02-ai-arena/src/FighterFarm.sol::324 => uint256(keccak256(abi.encode(msg.sender, fighters.length))),
2024-02-ai-arena/src/FighterFarm.sol::379 => uint256 dna = uint256(keccak256(abi.encode(msg.sender, tokenId, numRerolls[tokenId])));
2024-02-ai-arena/src/Neuron.sol::28 => bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
2024-02-ai-arena/src/Neuron.sol::31 => bytes32 public constant SPENDER_ROLE = keccak256("SPENDER_ROLE");
2024-02-ai-arena/src/Neuron.sol::34 => bytes32 public constant STAKER_ROLE = keccak256("STAKER_ROLE");
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Long Revert Strings

#### Impact
Issue Information: [G007](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g007---long-revert-strings)

#### Findings:
```
2024-02-ai-arena/src/AiArenaHelper.sol::17 => string[] public attributes = ["head", "eyes", "mouth", "body", "hands", "feet"];
2024-02-ai-arena/src/AiArenaHelper.sol::133 => require(probabilities.length == 6, "Invalid number of attribute arrays");
2024-02-ai-arena/src/FighterFarm.sol::9 => import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
2024-02-ai-arena/src/FighterFarm.sol::10 => import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
2024-02-ai-arena/src/FighterFarm.sol::396 => return "ipfs://bafybeifztjs4yuwhqi7bvzhw2ufksynkoiwxss2gnti6j4v25l7iwz7y44";
2024-02-ai-arena/src/GameItems.sol::5 => import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
2024-02-ai-arena/src/GameItems.sol::250 => return "ipfs://bafybeih3witscmml3padf4qxbea5jh4rl2xp67aydqvqsxmyuzipwtpnii";
2024-02-ai-arena/src/MergingPool.sol::196 => require(msg.sender == _rankedBattleAddress, "Not Ranked Battle contract address");
2024-02-ai-arena/src/Neuron.sol::4 => import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
2024-02-ai-arena/src/Neuron.sol::5 => import "@openzeppelin/contracts/access/AccessControl.sol";
2024-02-ai-arena/src/Neuron.sol::141 => "ERC20: claim amount exceeds allowance"
2024-02-ai-arena/src/Neuron.sol::156 => require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
2024-02-ai-arena/src/Neuron.sol::157 => require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
2024-02-ai-arena/src/Neuron.sol::174 => "ERC20: must have spender role to approve spending"
2024-02-ai-arena/src/Neuron.sol::187 => "ERC20: must have staker role to approve staking"
2024-02-ai-arena/src/Neuron.sol::199 => "ERC20: burn amount exceeds allowance"
2024-02-ai-arena/src/RankedBattle.sol::248 => require(hasUnstaked[tokenId][roundId] == false, "Cannot add stake after unstaking this round");
2024-02-ai-arena/src/RankedBattle.sol::295 => require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
2024-02-ai-arena/src/StakeAtRisk.sol::94 => require(msg.sender == _rankedBattleAddress, "Call must be from RankedBattle contract");
2024-02-ai-arena/src/StakeAtRisk.sol::97 => "Fighter does not have enough stake at risk"
2024-02-ai-arena/src/StakeAtRisk.sol::122 => require(msg.sender == _rankedBattleAddress, "Call must be from RankedBattle contract");
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Use Shift Right/Left instead of Division/Multiplication if possible

#### Impact
Issue Information: [G008](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md/#g008---use-shift-rightleft-instead-of-divisionmultiplication-if-possible)

#### Findings:
```
2024-02-ai-arena/src/Neuron.sol::37 => uint256 public constant INITIAL_TREASURY_MINT = 10**18 * 10**8 * 2;
2024-02-ai-arena/src/Neuron.sol::40 => uint256 public constant INITIAL_CONTRIBUTOR_MINT = 10**18 * 10**8 * 5;
2024-02-ai-arena/src/RankedBattle.sol::439 => curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Unsafe ERC20 Operation(s)

#### Impact
Issue Information: [L001](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l001---unsafe-erc20-operations)

#### Findings:
```
2024-02-ai-arena/src/FighterFarm.sol::376 => bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);
2024-02-ai-arena/src/GameItems.sol::164 => bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, price);
2024-02-ai-arena/src/RankedBattle.sol::251 => bool success = _neuronInstance.transferFrom(msg.sender, address(this), amount);
2024-02-ai-arena/src/RankedBattle.sol::283 => bool success = _neuronInstance.transfer(msg.sender, amount);
2024-02-ai-arena/src/RankedBattle.sol::493 => bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
2024-02-ai-arena/src/StakeAtRisk.sol::100 => bool success = _neuronInstance.transfer(_rankedBattleAddress, nrnToReclaim);
2024-02-ai-arena/src/StakeAtRisk.sol::143 => return _neuronInstance.transfer(treasuryAddress, totalStakeAtRisk[roundId]);
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Unspecific Compiler Version Pragma

#### Impact
Issue Information: [L003](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l003---unspecific-compiler-version-pragma)

#### Findings:
```
2024-02-ai-arena/src/AiArenaHelper.sol::2 => // pragma solidity ^0.8.20;
2024-02-ai-arena/src/AiArenaHelper.sol::3 => pragma solidity >=0.8.0 <0.9.0;
2024-02-ai-arena/src/FighterFarm.sol::2 => pragma solidity >=0.8.0 <0.9.0;
2024-02-ai-arena/src/GameItems.sol::2 => pragma solidity >=0.8.0 <0.9.0;
2024-02-ai-arena/src/MergingPool.sol::2 => pragma solidity >=0.8.0 <0.9.0;
2024-02-ai-arena/src/Neuron.sol::2 => pragma solidity >=0.8.0 <0.9.0;
2024-02-ai-arena/src/RankedBattle.sol::2 => pragma solidity >=0.8.0 <0.9.0;
2024-02-ai-arena/src/StakeAtRisk.sol::2 => pragma solidity >=0.8.0 <0.9.0;
2024-02-ai-arena/src/VoltageManager.sol::2 => pragma solidity >=0.8.0 <0.9.0;
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Do not use Deprecated Library Functions

#### Impact
Issue Information: [L005](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l005---do-not-use-deprecated-library-functions)

#### Findings:
```
2024-02-ai-arena/src/Neuron.sol::95 => _setupRole(MINTER_ROLE, newMinterAddress);
2024-02-ai-arena/src/Neuron.sol::103 => _setupRole(STAKER_ROLE, newStakerAddress);
2024-02-ai-arena/src/Neuron.sol::111 => _setupRole(SPENDER_ROLE, newSpenderAddress);
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)
