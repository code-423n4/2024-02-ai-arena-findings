
## Gas Optimizations

| Number | Issue | Instances |
|--------|-------|-----------|
|[G-01]| Precompute Off-Chain When Possible | 1 |
|[G-02]| Use uint8 Can Increase Gas Cost | 39 |
|[G-03]| Store Data in calldata Instead of Memory for Certain Function Parameters  | 11 |
|[G-04]| use Unchecked Blocks: | 41 |
|[G-05]| Use the existing Local variable/global variable when equal to a state variable to avoid reading from state | 3 |
|[G-06]| Avoid Assigning a Value that Will Never be Used | 12 |
|[G-07]| Check Arguments Early | 5 |
|[G-08]| State variables that could be declared constant | 2 |
|[G-09]| Inefficient assignments to state variables | 20 |
|[G-10]| Using Storage Instead of Memory for structs/arrays Saves Gas | 5 |
|[G-11]| Use assembly to write address storage values | 31 |
|[G-12]| Redundant state variable getters | 4 |
|[G-13]| Refactor functions such that all checks are performed before assigni-ng values to state variables | 1 |
|[G-14]| Refactor functions to combine events to reduce gas cost | 1 |
|[G-15]| Avoid Unnecessary Use of Storage | 2 |
|[G-16]| Using assembly to revert with an error message | 22 |
|[G-17]| Don’t cache if used only once | 3 |
|[G-18]| Switching between 1 and 2 instead of 0 and 1 (or false and true) is more gas efficient | 12 |
|[G-19]| aching global variables is more expensive than using the actual variable (use msg.sender instead of caching it) | 1 |
|[G-20]| Expensive operation inside a for loop | 4 |
|[G-21]| Use Mappings Instead of Arrays | 4 |
|[G-22]| Do not Calculate constant | 3 |
|[G-22]| Assigning keccak operations to constant variables results in extra gas costs | 3 |

## [G-01] Precompute Off-Chain When Possible

Computing values in advance outside the blockchain is cheaper than doing it inside smart contracts:

```solidity
// On-chain computation
function verify(uint numA, uint numB) {
  require(numA * numB < 1000);
}

// Precomputed off-chain
function verify(uint result) {
  require(result < 1000); 
}
```

Do computations like hashes, signatures outside.
Pass in precomputed values to save on-chain gas.

```solidity
file:  blob/main/src/RankedBattle.sol

341     uint256 stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
        if (amountStaked[tokenId] + stakeAtRisk > 0) {
            _addResultPoints(battleResult, tokenId, eloFactor, mergingPortion, fighterOwner);
        }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L341-L344

Before:
[PASS] testUpdateBattleRecordPlayerLossBattle() (gas: 854453)
[PASS] testUpdateBattleRecordPlayerTiedBattle() (gas: 754196)
[PASS] testUpdateBattleRecordPlayerWonBattle() (gas: 958706)

After:
[PASS] testUpdateBattleRecordPlayerLossBattle() (gas: 854447)
[PASS] testUpdateBattleRecordPlayerTiedBattle() (gas: 754190)
[PASS] testUpdateBattleRecordPlayerWonBattle() (gas: 958700)


## [G-02] Use uint8 Can Increase Gas Cost

A smart contract's gas consumption can be higher if developers use items that are less than 32 bytes in size because the Ethereum Virtual Machine can only handle 32 bytes at a time. In order to increase the element's size to the necessary size, the EVM has to perform additional operations.

```solidity
contract A { uint8 a = 0; }

```
The cost in the above example is 22,150 + 2,000 gas, compared with 7,050 gas when using a type higher than 32 bytes.

```solidity
contract A { uint a = 0; // or uint256 }

```

Only when you’re working with storage values is it advantageous to utilize reduced-size parameters because the compiler will compress several elements into one storage slot, combining numerous reads or writes into a single operation.

Smaller-size unsigned integers, such as uint8, are only more effective when multiple variables can be stored in the same storage space, like in structs. Uint256 uses less gas than uint8 in loops and other situations.


```solidity
file: blob/main/src/AiArenaHelper.sol

68    function addAttributeDivisor(uint8[] memory attributeDivisors) external {
        require(msg.sender == _ownerAddress);
        require(attributeDivisors.length == attributes.length);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeToDnaDivisor[attributes[i]] = attributeDivisors[i];
        }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L68-L75 

test only the function addAttributeDivisor();
Before 
[PASS] testAddAttributeDivisorFromOwner() (gas: 58249)

After 
[PASS] testAddAttributeDivisorFromOwner() (gas: 58051)


```solidity
file: blob/main/src/AiArenaHelper.sol

20   uint8[] public defaultAttributeDivisor = [2, 3, 5, 7, 11, 13];

30    mapping(uint256 => mapping(string => uint8[])) public attributeProbabilities;

33   mapping(string => uint8) public attributeToDnaDivisor;

41   constructor(uint8[][] memory probabilities) {

48   for (uint8 i = 0; i < attributesLength; i++) {

85   uint8 generation, 

86   uint8 iconsType, 

99   for (uint8 i = 0; i < attributesLength; i++) {

135  for (uint8 i = 0; i < attributesLength; i++) {

144  function deleteAttributeProbabilities(uint8 generation) public {

148  for (uint8 i = 0; i < attributesLength; i++) {

149  attributeProbabilities[generation][attributes[i]] = new uint8[](0);

160  returns (uint8[] memory) 

174  uint8[] memory attrProbabilities = getAttributeProbabilities(generation, attribute);

178  for (uint8 i = 0; i < attrProbabilitiesLength; i++) {

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L20


```solidity
file:  blob/main/src/FighterFarm.sol

32    uint8 public constant MAX_FIGHTERS_ALLOWED = 10;

35    uint8[2] public maxRerollsAllowed = [3, 3];

41    uint8[2] public generation = [0, 0];

78    mapping(uint256 => uint8) public numRerolls;

84    mapping(uint8 => uint8) public numElements;

87    mapping(uint8 => uint8) public numElements;

130   function incrementGeneration(uint8 fighterType) external returns (uint8) {

198   uint8[2] calldata numToMint,

246   uint8[] calldata fighterTypes,

247   uint8[] calldata iconsTypes,

374   function reRoll(uint8 tokenId, uint8 fighterType) public {

504   uint8 fighterType,

505   uint8 iconsType,

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L32


```solidity
file: blob/main/src/FighterOps.sol

18   uint8 generation

43   uint8 generation;

44   uint8 iconsType;

57   uint8 generation

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L18

```solidity
file: blob/main/src/RankedBattle.sol

53   uint8 public constant VOLTAGE_COST = 10;

325  uint8 battleResult,

417  uint8 battleResult, 

505  function _updateRecord(uint256 tokenId, uint8 battleResult) private {

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L53


```solidity
file: blob/main/src/VoltageManager.sol

16   event VoltageRemaining(address spender, uint8 voltage);  

39   mapping(address => uint8) public ownerVoltage;

105  function spendVoltage(address spender, uint8 voltageSpent) public {

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L16


## [G-03] Store Data in calldata Instead of Memory for Certain Function Parameters 

Instead of copying variables to memory, it is typically more cost-effective to load them immediately from calldata. If all you need to do is read data, you can conserve gas by saving the data in calldata.


```solidity
file: blob/main/src/AiArenaHelper.sol

68  function addAttributeDivisor(uint8[] memory attributeDivisors) external {

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L68


test only addAttributeProbabilities() function

Before: 
[PASS] testAddAttributeDivisorFromNonOwner() (gas: 16970)
[PASS] testAddAttributeDivisorFromOwner() (gas: 58249)

After:
[PASS] testAddAttributeDivisorFromNonOwner() (gas: 15922)
[PASS] testAddAttributeDivisorFromOwner() (gas: 57935)


```solidity
file: blob/main/src/AiArenaHelper.sol

41    constructor(uint8[][] memory probabilities) {

131   function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {

157   function getAttributeProbabilities(uint256 generation, string memory attribute) 

169   function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute) 

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L41

```solidity
file: blob/main/src/FighterFarm.sol

502  string memory modelHash,

503  string memory modelType,

506  uint256[2] memory customAttributes

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L506

```solidity
file: blob/main/src/GameItems.sol

209   string memory name_,

210   string memory tokenURI,

296   bytes memory data

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L209

## [G-04] use Unchecked Blocks:

Before Solidity 0.8.0, arithmetic operations would always wrap in case of underflow or overflow leading to the widespread use of libraries that introduce additional checks. Since Solidity 0.8.0, all arithmetic operations revert on overflow and underflow by default, but this behavior also includes some checks that cause the gas.

An unchecked block was introduced in Solidity that bypasses certain safety checks performed by the Solidity compiler. It is typically used when the developer wants to optimize the gas usage of their smart contract. However, using unchecked blocks also carries risks and by bypassing certain safety checks, the developer is taking on additional responsibility to ensure the code is secure and not vulnerable to attacks. The following functions are executed at the optimization of 200 runs:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.16;

contract WithoutUnchecked {
    uint256 private _value = 0;
    // function execution cost = 22322 gas
    function increment() public {
        _value += 1;
    }
}

contract WithUnchecked {
    uint256 private _value = 0;
    // function execution cost = 22225 gas
    function increment() public {
        unchecked {
            _value += 1;
        }
    }
}
```
As you can see using the unchecked block saved us 22322–22225 = 97 gas but only use this unchecked block if you are sure that the value will not overflow or underflow.


```solidity
file: blob/main/src/FighterFarm.sol

130  function incrementGeneration(uint8 fighterType) external returns (uint8) {
        require(msg.sender == _ownerAddress);
        generation[fighterType] += 1;
        maxRerollsAllowed[fighterType] += 1;
        return generation[fighterType];
    }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L130-L135

test only the incrementGeneration() function

Before 
[PASS] testIncrementGenerationFromOwner() (gas: 38454)

After
[PASS] testIncrementGenerationFromOwner() (gas: 37941)

Recomandation 

```solidity
 function incrementGeneration(uint8 fighterType) external returns (uint8) {
        require(msg.sender == _ownerAddress);
        unchecked {
            generation[fighterType] += 1;
            maxRerollsAllowed[fighterType] += 1;
        }

        return generation[fighterType];
    }

```

```solidity
file: blob/main/src/GameItems.sol

234   _itemCount += 1;

169   allowanceRemaining[msg.sender][tokenId] -= quantity;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L234

```solidity
file: blob/main/src/FighterFarm.sol
 
220  nftsClaimed[msg.sender][0] += numToMint[0];

221  nftsClaimed[msg.sender][1] += numToMint[1];

305  numTrained[tokenId] += 1;

306  totalNumTrained += 1;

389  numRerolls[tokenId] += 1;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L220


```solidity
file: blob/main/src/AiArenaHelper.sol
 
179   cumProb += attrProbabilities[i];

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L179

```solidity
file: blob/main/src/MergingPool.sol

126  totalPoints -= fighterPoints[winners[i]];

131  roundId += 1;

150  numRoundsClaimed[msg.sender] += 1;

160  claimIndex += 1;

180  numRewards += 1;

197  fighterPoints[tokenId] += points;

198  totalPoints += points;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L131

```solidity
file: blob/main/src/RankedBattle.sol

236  roundId += 1;

256   amountStaked[tokenId] += amount;

257   globalStakedAmount += amount;

275   amountStaked[tokenId] -= amount;

276   globalStakedAmount -= amount;

304   numRoundsClaimed[msg.sender] += 1;

307   amountClaimed[msg.sender] += claimableNRN;

348   totalBattles += 1;

450   points -= mergingPoints;

462   amountStaked[tokenId] += curStakeAtRisk;

466   accumulatedPointsPerFighter[tokenId][roundId] += points;

467   accumulatedPointsPerAddress[fighterOwner][roundId] += points;

468   totalAccumulatedPoints[roundId] += points;

485   accumulatedPointsPerFighter[tokenId][roundId] -= points;

486   accumulatedPointsPerAddress[fighterOwner][roundId] -= points;

487   totalAccumulatedPoints[roundId] -= points;

506  if (battleResult == 0) {
            fighterBattleRecord[tokenId].wins += 1;
        } else if (battleResult == 1) {
            fighterBattleRecord[tokenId].ties += 1;
        } else if (battleResult == 2) {
            fighterBattleRecord[tokenId].loses += 1;
        }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L236

```solidity
file: blob/main/src/StakeAtRisk.sol

102  stakeAtRisk[roundId][fighterId] -= nrnToReclaim;

105  totalStakeAtRisk[roundId] -= nrnToReclaim;

104  amountLost[fighterOwner] -= nrnToReclaim;

123  stakeAtRisk[roundId][fighterId] += nrnToPlaceAtRisk;

124  totalStakeAtRisk[roundId] += nrnToPlaceAtRisk;

125  amountLost[fighterOwner] += nrnToPlaceAtRisk;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L123

## [G-05] Use the existing Local variable/global variable when equal to a state variable to avoid reading from state

Local variable lowerBound should be used instead of reading numRoundsClaimed[msg.sender] from storage

```solidity
file:  blob/main/src/RankedBattle.sol

298     uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            nrnDistribution = getNrnDistribution(currentRound);
            claimableNRN += (
                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
            ) / totalAccumulatedPoints[currentRound];
            numRoundsClaimed[msg.sender] += 1;
        }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L298-L305



Before 
[PASS] testClaimNRN() (gas: 1781216)

After:
[PASS] testClaimNRN() (gas: 1740492)

use like this 

```solidity

298     uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            nrnDistribution = getNrnDistribution(currentRound);
            claimableNRN += (
                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
            ) / totalAccumulatedPoints[currentRound];
            lowerBound  += 1;
        }

```
Local variable lowerBound should be used instead of reading numRoundsClaimed[msg.sender] from storage


```solidity
file: blob/main/src/MergingPool.sol

148          uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            numRoundsClaimed[msg.sender] += 1;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L148-L151

## [G-06] Avoid Assigning a Value that Will Never be Used

In Solidity, every variable assignment has an associated gas cost. When initializing variables, default values are often assigned that may never be used, resulting in wasted gas. So avoid assigning the default values that are not used.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.7;

contract withAssignment {
    // gas cost = 21,993
    function getVal(uint256 x) external returns (uint256) {
        uint256 val = 0;
        val = val + x;
        return val;
    }
}

contract withoutAssignment {
    // gas cost = 21,985
    function getVal(uint256 x) external returns (uint256) {
        uint256 val;
        val = val + x;
        return val;
    }
}
```

```solidity
file: blob/main/src/AiArenaHelper.sol

176   uint256 cumProb = 0;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L178


```solidity
file: blob/main/src/GameItems.sol

64   uint256 _itemCount = 0;    

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L64

```solidity
file: blob/main/src/MergingPool.sol

29   uint256 public roundId = 0;

32   uint256 public totalPoints = 0;

147  uint32 claimIndex = 0;

174  uint256 numRewards = 0;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L29


```solidity
file: blob/main/src/RankedBattle.sol

56   uint256 public totalBattles = 0;

59   uint256 public globalStakedAmount = 0;

62   uint256 public roundId = 0;

296  uint256 claimableNRN = 0;

387  uint256 claimableNRN = 0;

427  uint256 points = 0;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L56

## [G-07] Check Arguments Early

Checks that require() or revert() statements that check input arguments are at the top of the function. Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a gas load in a function that may ultimately revert in the unhappy case.

```solidity
file: blob/main/src/AiArenaHelper.sol

133  require(probabilities.length == 6, "Invalid number of attribute arrays");

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L133

```solidity
file: blob/main/src/FighterFarm.sol

373  require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L373

```solidity
file: blob/main/src/GameItems.sol

150   require(_neuronInstance.balanceOf(msg.sender) >= price, "Not enough NRN for purchase");

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L150

```solidity
file: blob/main/src/MergingPool.sol

120  require(winners.length == winnersPerPeriod, "Incorrect number of winners");

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L120

```solidity
file: blob/main/src/StakeAtRisk.sol

95  require(
            stakeAtRisk[roundId][fighterId] >= nrnToReclaim, 
            "Fighter does not have enough stake at risk"
        );

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L95-L98

## [G-08] State variables that could be declared constant

the instence are not find by bot

State variables that are not updated following deployment should be declared constant to save gas.

```solidity
file: blob/main/src/MergingPool.sol

26  uint256 public winnersPerPeriod = 2;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L26

```solidity
file: blob/main/src/RankedBattle.sol

66  uint256 public bpsLostPerLoss = 10;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L66

## [G-09] Inefficient assignments to state variables

Assignment operations that involve calculations, like add-assigment (+=), for state variables, should be replaced with the appropriate calculation operation, like add (+), followed by an assignment operation (=), to save gas. Avoids a Gwarmaccess (100 gas).

```solidity
file: blob/main/src/FighterFarm.sol
 
294  totalNumTrained += 1;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L294

```solidity
file: blob/main/src/GameItems.sol

234  _itemCount += 1;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L234

```solidity
file: blob/main/src/MergingPool.sol

131  roundId += 1;

198  totalPoints += points;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L131

```solidity
file: blob/main/src/RankedBattle.sol
 
236  roundId += 1;

256  amountStaked[tokenId] += amount;

257  globalStakedAmount += amount;

304  numRoundsClaimed[msg.sender] += 1;

307  amountClaimed[msg.sender] += claimableNRN;

348  totalBattles += 1;

462  amountStaked[tokenId] += curStakeAtRisk;

466  accumulatedPointsPerFighter[tokenId][roundId] += points;

467  accumulatedPointsPerAddress[fighterOwner][roundId] += points;

468  totalAccumulatedPoints[roundId] += points;

507  fighterBattleRecord[tokenId].wins += 1;

509  fighterBattleRecord[tokenId].ties += 1;

511  fighterBattleRecord[tokenId].loses += 1;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L236

```solidity
file: blob/main/src/StakeAtRisk.sol

123  stakeAtRisk[roundId][fighterId] += nrnToPlaceAtRisk;

124  totalStakeAtRisk[roundId] += nrnToPlaceAtRisk;

125  amountLost[fighterOwner] += nrnToPlaceAtRisk;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L123

## [G-10] Using Storage Instead of Memory for structs/arrays Saves Gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct / array to be read from storage. This incurs a Gcoldsload (2100 gas) for each field of the struct / array. If the fields are read from the new memory variable, they incur an additional MLOAD, which is more expensive than a simple stack read.

Instead of declaring the variable with the memory keyword, a more cost-effective approach is to declare the variable with the storage keyword. In this case, you can cache any fields that need to be re-read in stack variables. This approach significantly reduces gas costs, as you only incur the Gcoldsload for the fields actually read.

The only scenario where it makes sense to read the whole struct / array into a memory variable is if the full struct / array is being returned by the function or passed to a function that requires memory. Additionally, if the array / struct is being read from another memory array / struct, using a memory variable may be appropriate.

By carefully considering the storage and memory usage in your contract, you can optimize gas costs and improve the efficiency of your smart contract operations.

```solidity
file: blob/main/src/FighterFarm.sol

510  FighterOps.FighterPhysicalAttributes memory attrs = _aiArenaHelperInstance.createPhysicalAttributes(

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L510


```solidity
file: blob/main/src/AiArenaHelper.sol

96   uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributes.length);

174  uint8[] memory attrProbabilities = getAttributeProbabilities(generation, attribute);

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L96

```solidity
file: blob/main/src/MergingPool.sol

123  address[] memory currentWinnerAddresses = new address[](winnersLength);

206  uint256[] memory points = new uint256[](1);

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L123

## [G-11] Use assembly to write address storage values

Using assembly to write address storage values can potentially save gas costs.
This detector flags instances where address storage values are written without using assembly.

```solidity
file: blob/main/src/AiArenaHelper.sol

63  _ownerAddress = newOwnerAddress;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L63


```solidity
file: blob/main/src/StakeAtRisk.sol

65    treasuryAddress = treasuryAddress_;

66    _rankedBattleAddress = rankedBattleAddress; 

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L65

```solidity
file: blob/main/src/VoltageManager.sol
 
52  _ownerAddress = ownerAddress;

66  _ownerAddress = newOwnerAddress;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L52

```solidity
file: blob/main/src/FighterFarm.sol

107    _ownerAddress = ownerAddress;

108    _delegatedAddress = delegatedAddress;

109    treasuryAddress = treasuryAddress_;

122    _ownerAddress = newOwnerAddress;

157    _mintpassInstance = AAMintPass(mintpassAddress);

165   _neuronInstance = Neuron(neuronAddress);

173   _mergingPoolAddress = mergingPoolAddress;

``` 
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L107

```solidity
file:  blob/main/src/GameItems.sol

96    _ownerAddress = ownerAddress;

97     treasuryAddress = treasuryAddress_;

110   _ownerAddress = newOwnerAddress;

141   _neuronInstance = Neuron(nrnAddress);

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L96\

```solidity
file: blob/main/src/MergingPool.sol

76    _ownerAddress = ownerAddress;

77    _rankedBattleAddress = rankedBattleAddress;

78    _fighterFarmInstance = FighterFarm(fighterFarmAddress);

91   _ownerAddress = newOwnerAddress;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L76

```solidity
file: blob/main/src/Neuron.sol

71   _ownerAddress = ownerAddress;

72    treasuryAddress = treasuryAddress_;

87   _ownerAddress = newOwnerAddress;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L71

```solidity
file:  blob/main/src/RankedBattle.sol

152    _ownerAddress = ownerAddress;

153    _gameServerAddress = gameServerAddress;

154    _fighterFarmInstance = FighterFarm(fighterFarmAddress);

155    _voltageManagerInstance = VoltageManager(voltageManagerAddress);

169    _ownerAddress = newOwnerAddress;

186    _gameServerAddress = gameServerAddress;
 
194    _stakeAtRiskAddress = stakeAtRiskAddress;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L152

## [G-12] Redundant state variable getters

Getters for public state variables are automatically generated by the solidity compiler so
there is no need to code them manually as this increases deployment cost.

### Make rankedNrnDistribution mapping variable private or internal since a getter function was defined for it.

The solidity compiler would automatically create a getter function for the rankedNrnDistribution mapping since
it is declared as a public variable but a getter function getNrnDistribution() was also declared
in the contract for the same variable thereby making it two getter functions for the same variable
in the contract. We could rectify this issue by making the rankedNrnDistribution variable private or
internal (if the variable is to be inherited). The diff below shows how the code could be refactored:

```solidity
file: blob/main/src/RankedBattle.sol

122  mapping(uint256 => uint256) public rankedNrnDistribution;
.
.
.
.
379  function getNrnDistribution(uint256 roundId_) public view returns(uint256) {
        return rankedNrnDistribution[roundId_];
    }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L379-L381

```solidity
122  - mapping(uint256 => uint256) public rankedNrnDistributio

122  + mapping(uint256 => uint256) internal rankedNrnDistributio

```

### Make stakeAtRisk mapping variable private or internal since a getter function was defined for it.

The solidity compiler would automatically create a getter function for the stakeAtRisk mapping since
it is declared as a public variable but a getter function getStakeAtRisk() was also declared
in the contract for the same variable thereby making it two getter functions for the same variable
in the contract. We could rectify this issue by making the stakeAtRisk variable private or
internal (if the variable is to be inherited). The diff below shows how the code could be refactored:

```solidity
file: blob/main/src/StakeAtRisk.sol

46  mapping(uint256 => mapping(uint256 => uint256)) public stakeAtRisk;
.
.
.
132  function getStakeAtRisk(uint256 fighterId) external view returns(uint256) {
        return stakeAtRisk[roundId][fighterId];
    }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L132-L134

```solidity
122  - mapping(uint256 => mapping(uint256 => uint256)) public stakeAtRisk;

122  + mapping(uint256 => mapping(uint256 => uint256)) internal stakeAtRisk;

```

### Make attributeProbabilities mapping variable private or internal since a getter function was defined for it.

```solidity
file: blob/main/src/AiArenaHelper.sol

30   mapping(uint256 => mapping(string => uint8[])) public attributeProbabilities;
.
.
.
.
157  function getAttributeProbabilities(uint256 generation, string memory attribute) 
        public 
        view 
        returns (uint8[] memory) 
    {
        return attributeProbabilities[generation][attribute];
    }  

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L157-L164

### Make allowanceRemaining mapping variable private or internal since a getter function was defined for it.


```solidity
file: blob/main/src/GameItems.sol

74  mapping(address => mapping(uint256 => uint256)) public allowanceRemaining;
.
.
.
.
268  function getAllowanceRemaining(address owner, uint256 tokenId) public view returns (uint256) {
        uint256 remaining = allowanceRemaining[owner][tokenId];
        if (dailyAllowanceReplenishTime[owner][tokenId] <= block.timestamp) {
            remaining = allGameItemAttributes[tokenId].dailyAllowance;
        }
        return remaining;
    }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L268-L274

## [G-13] Refactor functions such that all checks are performed before assigning values to state variables

Refactor the updateFighterStaking() function of FighterFarm contract such that all 
the checks on the arguments are performed before assigning the values to state variables.

```solidity
file: blob/main/src/FighterFarm.sol

268  function updateFighterStaking(uint256 tokenId, bool stakingStatus) external {
        require(hasStakerRole[msg.sender]);
        fighterStaked[tokenId] = stakingStatus;
        if (stakingStatus) {
            emit Locked(tokenId);
        } else {
            emit Unlocked(tokenId);
        }
    }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L271

## [G-14] Refactor functions to combine events to reduce gas cost

In scenarios where a function emits multiple event but these events are only emitted by that function we can combine these events to a single event in doing this we would save at least 1 Glog0 375 gas units

The FighterFarm.updateFighterStaking() function emits two events Locked and Unlocked but these events are only emitted in this function. We can make the FighterFarm.updateFighterStaking() function more gas efficient if we combine these Locked and Unlocked events to a single event. The diff below shows how the code could be refactored:

```solidity
file: blob/main/src/FighterFarm.sol

279  function updateFighterStaking(
        uint256 tokenId,
        bool stakingStatus
    ) external {
        require(hasStakerRole[msg.sender]);
        fighterStaked[tokenId] = stakingStatus;
        if (stakingStatus) {
            emit Locked(tokenId);
        } else {
            emit Unlocked(tokenId);
        }
    }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L279-L290

## [G-15] Avoid Unnecessary Use of Storage

Writing data to storage is one of the most expensive operations in Solidity. Reduce gas costs by avoiding unnecessary writes. For example, move constant values to memory:

```solidity

// Expensive
string public constant NAME = "MyContract"; 

// Cheaper
string memory NAME = "MyContract";

```

Also be careful about storing large pieces of data on-chain. Use alternative patterns like IPFS for larger data.

Storing data costs 20,000 gas versus 3 gas for memory.
Minimize storage by using memory for fixed constants and temporary values.
Store large data like files off-chain (IPFS) and put hash on-chain.

```solidity
file: blob/main/src/GameItems.sol

49  string public name = "AI Arena Game Items";

52   string public symbol = "AGI";

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L49

## [G-16] Using assembly to revert with an error message

Issue Description - When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

Estimated Gas Savings - Here’s an example:

```solidity

/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;
    constructor() {
        owner = msg.sender;
    }
    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}
/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;
    constructor() {
        owner = msg.sender;
    }
    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}

```
From the example above, we can see that we get a gas saving of over 300 gas when reverting with the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

Code Snippets:

```solidity
file: blob/main/src/AiArenaHelper.sol

133  require(probabilities.length == 6, "Invalid number of attribute arrays");

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L133

```solidity
file: blob/main/src/GameItems.sol

150  require(_neuronInstance.balanceOf(msg.sender) >= price, "Not enough NRN for purchase");

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L150


```solidity
file: blob/main/src/MergingPool.sol

120  require(winners.length == winnersPerPeriod, "Incorrect number of winners");

121  require(!isSelectionComplete[roundId], "Winners are already selected");

196  require(msg.sender == _rankedBattleAddress, "Not Ranked Battle contract address");

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L120

```solidity
file: blob/main/src/RankedBattle.sol

245   require(amount > 0, "Amount cannot be 0");

246   require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");

247   require(_neuronInstance.balanceOf(msg.sender) >= amount, "Stake amount exceeds balance");

248   require(hasUnstaked[tokenId][roundId] == false, "Cannot add stake after unstaking this round");

271   require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");

295   require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L245

```solidity
file: blob/main/src/Neuron.sol

156   require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");

157   require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");

139  require(
            allowance(treasuryAddress, msg.sender) >= amount, 
            "ERC20: claim amount exceeds allowance"
        );
172  require(
            hasRole(SPENDER_ROLE, msg.sender), 
            "ERC20: must have spender role to approve spending"
        );
    
185   require(
            hasRole(STAKER_ROLE, msg.sender), 
            "ERC20: must have staker role to approve staking"
        );

197   require(
            allowance(account, msg.sender) >= amount, 
            "ERC20: burn amount exceeds allowance"
        );

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L156


```solidity
file: blob/main/src/StakeAtRisk.sol

79  require(msg.sender == _rankedBattleAddress, "Not authorized to set new round");

94  require(msg.sender == _rankedBattleAddress, "Call must be from RankedBattle contract");

95   require(
            stakeAtRisk[roundId][fighterId] >= nrnToReclaim, 
            "Fighter does not have enough stake at risk"
        );

122   require(msg.sender == _rankedBattleAddress, "Call must be from RankedBattle contract");
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L79

## [G-17] Don’t cache if used only once

Caching here will simply increase gas cost, since it doesn’t affect readability, we should not cache since we only reference the cached variables only once.

```solidity
file: blob/main/src/Neuron.sol

201  uint256 decreasedAllowance = allowance(account, msg.sender) - amount;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L201

```solidity
file: blob/main/src/RankedBattle.sol

341  uint256 stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L341


```solidity
file: blob/main/src/StakeAtRisk.sol

80   bool success = _sweepLostStake();

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L80

## [G-18] Switching between 1 and 2 instead of 0 and 1 (or false and true) is more gas efficient

SSTORE from 0 to 1 (or any non-zero value) costs 20000 gas. SSTORE from 1 to 2 (or any other non-zero value) costs 5000 gas.

By storing the original value once again, a refund is triggered (https://eips.ethereum.org/EIPS/eip-2200).

Since refunds are capped to a percentage of the total transaction’s gas, it is best to keep them low, to increase the likelihood of the full refund coming into effect.

Therefore, switching between 1, 2 instead of 0, 1 will be more gas efficient.



```solidity
file: blob/main/src/FighterFarm.sol

142  hasStakerRole[newStaker] = true;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L142


```solidity
file: blob/main/src/GameItems.sol

98   isAdmin[_ownerAddress] = true;

187   allowedBurningAddresses[newBurningAddress] = true;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L98

```solidity
file: blob/main/src/MergingPool.sol

79   isAdmin[_ownerAddress] = true;

130  isSelectionComplete[roundId] = true;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L79

```solidity
file: blob/main/src/Neuron.sol

73  isAdmin[_ownerAddress] = true;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L73

```solidity
file: blob/main/src/RankedBattle.sol

156  isAdmin[_ownerAddress] = true;

262  _calculatedStakingFactor[tokenId][roundId] = true;

281   _calculatedStakingFactor[tokenId][roundId] = true;

282   hasUnstaked[tokenId][roundId] = true;

435  _calculatedStakingFactor[tokenId][roundId] = true;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L156

```solidity
file: blob/main/src/VoltageManager.sol

53  isAdmin[_ownerAddress] = true;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L53


## [G-19] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

The msg.sender variable is a special variable that always refers to the address of the sender of the current transaction. This variable is not stored in memory, so it is much cheaper to use than a cached global variable.

```solidity
file: blob/main/src/AiArenaHelper.sol

42  _ownerAddress = msg.sender;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L42

## [G-20] Expensive operation inside a for loop

```solidity
file: blob/main/src/AiArenaHelper.sol

48  for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }

99   for (uint8 i = 0; i < attributesLength; i++) {
                if (
                  i == 0 && iconsType == 2 || // Custom icons head (beta helmet)
                  i == 1 && iconsType > 0 || // Custom icons eyes (red diamond)
                  i == 4 && iconsType == 3 // Custom icons hands (bowling ball)
                ) {
                    finalAttributeProbabilityIndexes[i] = 50;
                } else {
                    uint256 rarityRank = (dna / attributeToDnaDivisor[attributes[i]]) % 100;
                    uint256 attributeIndex = dnaToIndex(generation, rarityRank, attributes[i]);
                    finalAttributeProbabilityIndexes[i] = attributeIndex;
                }
            }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L48-L51


```solidity 
file: blob/main/src/MergingPool.sol

124  for (uint256 i = 0; i < winnersLength; i++) {
            currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);
            totalPoints -= fighterPoints[winners[i]];
            fighterPoints[winners[i]] = 0;
        }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L124-L128

```solidity
file: blob/main/src/RankedBattle.sol

390  for (uint32 i = lowerBound; i < roundId; i++) {
            nrnDistribution = getNrnDistribution(i);
            claimableNRN += (
                accumulatedPointsPerAddress[claimer][i] * nrnDistribution
            ) / totalAccumulatedPoints[i];
        }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L390-L395

## [G-21] Use Mappings Instead of Arrays

There are two data types to describe lists of data in Solidity, arrays and maps, and their syntax and structure are quite different, allowing each to serve a distinct purpose. While arrays are packable and iterable, mappings are less expensive.

For example, creating an array of cars in Solidity might look like this:

```solidity
string cars[];
cars = ["ford", "audi", "chevrolet"];

```
Let’s see how to create a mapping for cars:

```solidity
mapping(uint => string) public cars

```
When using the mapping keyword, you will specify the data type for the key (uint) and the value (string). Then you can add some data using the constructor function.

```solidity

constructor() public {
        cars[101] = "Ford";
        cars[102] = "Audi";
        cars[103] = "Chevrolet";
    }
}


```

Except where iteration is required or data types can be packed, it is advised to use mappings to manage lists of data in order to conserve gas. This is beneficial for both memory and storage.

An integer index can be used as a key in a mapping to control an ordered list. Another advantage of mappings is that you can access any value without having to iterate through an array as would otherwise be necessary. 

```solidity
file: blob/main/src/AiArenaHelper.sol

17   string[] public attributes = ["head", "eyes", "mouth", "body", "hands", "feet"];

20    uint8[] public defaultAttributeDivisor = [2, 3, 5, 7, 11, 13];

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L17


```solidity
file: blob/main/src/FighterFarm.sol

35    uint8[2] public maxRerollsAllowed = [3, 3];

41    uint8[2] public generation = [0, 0];

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L35

## [G-22] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time the variable is used, which wastes some gas.

```solidity
file: blob/main/src/Neuron.sol

37  uint256 public constant INITIAL_TREASURY_MINT = 10**18 * 10**8 * 2;

40  uint256 public constant INITIAL_CONTRIBUTOR_MINT = 10**18 * 10**8 * 5;

43  uint256 public constant MAX_SUPPLY = 10**18 * 10**9;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L37

## [G-23] Assigning keccak operations to constant variables results in extra gas costs

"constants" expressions are expressions. As such, keccak assigned to a constant variable are re-computed at each use of the variable, which will consume gas unnecessarily. To save gas, Changing the variable from constant to immutable will make the computation run only once and therefore save gas.

```solidity
file:  blob/main/src/Neuron.sol

28  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

31  bytes32 public constant SPENDER_ROLE = keccak256("SPENDER_ROLE");

34  bytes32 public constant STAKER_ROLE = keccak256("STAKER_ROLE");

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L28