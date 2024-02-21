# Gas report by 0xDetermination

|Issue number|Description|
|:-|:-|
|[G-1](#g-1)|`Neuron.claim()` allowance check is not necessary because `transferFrom()` also checks allowance|
|[G-2](#g-2)|`Neuron.claim()` can call code in `transferFrom()` implementation directly instead of calling `transferFrom()`|
|[G-3](#g-3)|`Neuron.burnFrom()` allowance check is not necessary|
|[G-4](#g-4)|`RankedBattle._getStakingFactor()` accesses the same storage variable a second time whenever it's called (in `stakeNRN()`, `unstakeNRN()`, and `updateBattleRecord()`)|
|[G-5](#g-5)|NRN `success` checks are unnecessary because the OZ implementation will not fail without reverting|
|[G-6](#g-6)|`RankedBattle.claimNRN()` is gas inefficient for addresses that don't have any points in previous rounds|
|[G-7](#g-7)|`RankedBattle.updateBattleRecord()` doesn't need to check voltage since `VoltageManager.spendVoltage()` will revert if voltage is too low|
|[G-8](#g-8)|`RankedBattle._addResultPoints()` can put code inside an existing conditional statement to avoid incrementing storage variables by zero|
|[G-9](#g-9)|`RankedBattle._addResultPoints()` executes unnecessary lines of code in case of a tie|
|[G-10](#g-10)|Unnecessary equality comparison in `RankedBattle._updateRecord()`|
|[G-11](#g-11)|`FighterFarm.sol` inherits `ERC721Enumerable` but the codebase doesn't use any of the enumerable methods|
|[G-12](#g-12)|Storing variable types smaller than 32 bytes costs more gas than types with 32 bytes and doesn't save cold SLOAD gas in `FighterFarm.sol`|
|[G-13](#g-13)|Balance check in `FighterFarm.reRoll()` is not necessary since `transferFrom()` will revert in case of insufficient balance|
|[G-14](#g-14)|`FighterFarm._beforeTokenTransfer()` can be deleted to save an extra function call|
|[G-15](#g-15)|`FighterFarm._createFighterBase()` performs unnecessary check when minting dendroid|
|[G-16](#g-16)|`MergingPool.claimRewards()` should update `numRoundsClaimed` after the loop ends instead of incrementing it in every loop|
|[G-17](#g-17)|`MergingPool.pickWinner()` and `MergingPool.claimRewards()` can be refactored so that users don't have to iterate through the entire array of winners for every round|
|[G-18](#g-18)|DNA rarity calculation in `AiArenaHelper.dnaToIndex()` can be redesigned to save gas|
|[G-19](#g-19)|`AiArenaHelper.createPhysicalAttributes()` should check for `iconsType` before running icons-relevant code since icons fighters are rare|
|[G-20](#g-20)|`AiArenaHelper` constructor sets probabilities twice|


## [G-1]<a name="g-1"></a> `Neuron.claim()` allowance check is not necessary because `transferFrom()` also checks allowance
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L139-L142
### Description
The require statement below can be removed since `transferFrom()` will check/update the allowance.
```solidity
    function claim(uint256 amount) external {
        require(
            allowance(treasuryAddress, msg.sender) >= amount, 
            "ERC20: claim amount exceeds allowance"
        );
        transferFrom(treasuryAddress, msg.sender, amount);
        emit TokensClaimed(msg.sender, amount);
    }
```
### Recommended Fix
Delete the require statement.

## [G-2]<a name="g-2"></a> `Neuron.claim()` can call code in `transferFrom()` implementation directly instead of calling `transferFrom()`
### Links
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.9/contracts/token/ERC20/ERC20.sol#L158-L163
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L143
### Description
See the OZ `transferFrom()` implementation:
```solidity
    function transferFrom(address from, address to, uint256 amount) public virtual override returns (bool) {
        address spender = _msgSender();
        _spendAllowance(from, spender, amount);
        _transfer(from, to, amount);
        return true;
    }
```
`Neuron.claim()` can run the above code directly without spending extra gas for the function call and to cache `msg.sender`.
### Recommended Fix
```diff
    function claim(uint256 amount) external {
        require(
            allowance(treasuryAddress, msg.sender) >= amount, 
            "ERC20: claim amount exceeds allowance"
        );
-       transferFrom(treasuryAddress, msg.sender, amount);
+       _spendAllowance(treasuryAddress, msg.sender, amount);
+       _transfer(treasuryAddress, msg.sender, amount);
        emit TokensClaimed(msg.sender, amount);
    }
```

## [G-3]<a name="g-3"></a> `Neuron.burnFrom()` allowance check is not necessary
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L196-L204
### Description
Note below that if the amount exceeds the allowance, the `decreasedAllowance` calculation will underflow and cause a revert. The `require` allowance check can be removed.
```solidity
    function burnFrom(address account, uint256 amount) public virtual {
        require(
            allowance(account, msg.sender) >= amount, 
            "ERC20: burn amount exceeds allowance"
        );
        uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
        _burn(account, amount);
        _approve(account, msg.sender, decreasedAllowance);
    }
```
### Recommended Fix
Remove the require statement.

## [G-4]<a name="g-4"></a> `RankedBattle._getStakingFactor()` accesses the same storage variable a second time whenever it's called (in `stakeNRN()`, `unstakeNRN()`, and `updateBattleRecord()`)
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L528
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L253-L258
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L272-L277
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L342
### Description
`RankedBattle._getStakingFactor()` accesses the storage variable `amountStaked[tokenId]` after the three mentioned functions access the same variable (see links). This variable should be cached to decrease storage reads and gas costs.
### Recommended Fix
Cache `amountStaked[tokenId]` and pass it to `_getStakingFactor()`.

## [G-5]<a name="g-5"></a> NRN `success` checks are unnecessary because the OZ implementation will not fail without reverting
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L284
### Description
The pattern in the link provided (checking for NRN transfer success) is used widely in the codebase, but these checks are unnecessary since transfers won't fail without reverting.
### Recommended Fix
Remove the NRN `success` checks.

## [G-6]<a name="g-6"></a> `RankedBattle.claimNRN()` is gas inefficient for addresses that don't have any points in previous rounds
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L294-L311
### Description
The for loop in `claimNRN()` will start at zero for new users and spend thousands of gas accessing/setting storage in each loop, although the user will not have any rewards in earlier rounds.
```solidity
    function claimNRN() external {
        require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
        uint256 claimableNRN = 0;
        uint256 nrnDistribution;
        uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            nrnDistribution = getNrnDistribution(currentRound);
            claimableNRN += (
                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
            ) / totalAccumulatedPoints[currentRound];
            numRoundsClaimed[msg.sender] += 1;
        }
```
### Recommended Fix
Allow users to set their `numRoundsClaimed` to an arbitrary value.

## [G-7]<a name="g-7"></a> `RankedBattle.updateBattleRecord()` doesn't need to check voltage since `VoltageManager.spendVoltage()` will revert if voltage is too low
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L334-L338
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L110
### Description
`updateBattleRecord()` checks that the initiator's voltage is sufficient:
```solidity
        require(
            !initiatorBool ||
            _voltageManagerInstance.ownerVoltageReplenishTime(fighterOwner) <= block.timestamp || 
            _voltageManagerInstance.ownerVoltage(fighterOwner) >= VOLTAGE_COST
        );
```
This check is not necessary, since `VoltageManager.spendVoltage()` will revert if the voltage is too low:
```solidity
    function spendVoltage(address spender, uint8 voltageSpent) public {
        require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
        if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
            _replenishVoltage(spender);
        }
        ownerVoltage[spender] -= voltageSpent; // UNDERFLOWS AND REVERTS IF VOLTAGE IS TOO LOW
        emit VoltageRemaining(spender, ownerVoltage[spender]);
    }
```
Note that this check can also be performed offchain before calling `updateBattleRecord()`. 
### Recommended Fix
Remove the require statement.

## [G-8]<a name="g-8"></a> `RankedBattle._addResultPoints()` can put code inside an existing conditional statement to avoid incrementing storage variables by zero
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L466-L471
### Description
See the following code from the `RankedBattle._addResultPoints()` below. The `points` variable can be zero, such as if the player's `mergingFactor` is 100. The lines updating storage variables with `points` should be put inside the `if` block.
```solidity
            accumulatedPointsPerFighter[tokenId][roundId] += points;
            accumulatedPointsPerAddress[fighterOwner][roundId] += points;
            totalAccumulatedPoints[roundId] += points;
            if (points > 0) {
                emit PointsChanged(tokenId, points, true);
            }
```
### Recommended Fix
```diff
-           accumulatedPointsPerFighter[tokenId][roundId] += points;
-           accumulatedPointsPerAddress[fighterOwner][roundId] += points;
-           totalAccumulatedPoints[roundId] += points;
            if (points > 0) {
+               accumulatedPointsPerFighter[tokenId][roundId] += points;
+               accumulatedPointsPerAddress[fighterOwner][roundId] += points;
+               totalAccumulatedPoints[roundId] += points;
                emit PointsChanged(tokenId, points, true);
            }
```

## [G-9]<a name="g-9"></a> `RankedBattle._addResultPoints()` executes unnecessary lines of code in case of a tie
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L425-L439
### Description
`RankedBattle._addResultPoints()` runs all of the below code in case of a tie; it's all setup for the win/lose code blocks, which make up the rest of the function. There's no more code that runs in case of a tie. The only state modification in this function in case of a tie is updating the stakingFactor if it was not already updated, but since points/stake does not change in case of a tie the update is not necessary.
```solidity
        uint256 stakeAtRisk;
        uint256 curStakeAtRisk;
        uint256 points = 0;

        /// Check how many NRNs the fighter has at risk
        stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);

        /// Calculate the staking factor if it has not already been calculated for this round 
        if (_calculatedStakingFactor[tokenId][roundId] == false) {
            stakingFactor[tokenId] = _getStakingFactor(tokenId, stakeAtRisk);
            _calculatedStakingFactor[tokenId][roundId] = true;
        }

        /// Potential amount of NRNs to put at risk or retrieve from the stake-at-risk contract
        curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;
        // THE REST OF THE CODE IN THIS FUNCTION ONLY RUNS IN CASE OF A WIN OR LOSS
```
### Recommended Fix
Cut the code above and paste it at the top of both the win and lose blocks.

## [G-10]<a name="g-10"></a> Unnecessary equality comparison in `RankedBattle._updateRecord()`
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L505-L513
### Description
There are only three cases for `battleResult`, so below it's not necessary to check the result 3 times. An `else` block can replace the second `else if` block.
```solidity
    function _updateRecord(uint256 tokenId, uint8 battleResult) private {
        if (battleResult == 0) {
            fighterBattleRecord[tokenId].wins += 1;
        } else if (battleResult == 1) {
            fighterBattleRecord[tokenId].ties += 1;
        } else if (battleResult == 2) {
            fighterBattleRecord[tokenId].loses += 1;
        }
    }
```
### Recommended Fix
```diff
-       } else if (battleResult == 2) {
+       } else {
```


## [G-11]<a name="g-11"></a> `FighterFarm.sol` inherits `ERC721Enumerable` but the codebase doesn't use any of the enumerable methods
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L16
### Description
Solely inheriting `ERC721` in the `FighterFarm` contract would save on deployment costs since the codebase doesn't use the enumerable methods.
### Recommended Fix
If desired, don't inherit from `ERC721Enumerable`.

## [G-12]<a name="g-12"></a> Storing variable types smaller than 32 bytes costs more gas than types with 32 bytes and doesn't save cold SLOAD gas in `FighterFarm.sol`
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L76-L91
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L45
### Description
See below code for an example:
```solidity
    /// @notice Mapping to keep track of how many times an nft has been re-rolled.
    mapping(uint256 => uint8) public numRerolls;
```
The EVM will spend more gas to store a smaller type like `uint8` compared to a 32 byte type like `uint`.

See the links for more occurrences.
### Recommended Fix
Consider changing the mapped-to types to 32 byte types like `uint`.

## [G-13]<a name="g-13"></a> Balance check in `FighterFarm.reRoll()` is not necessary since `transferFrom()` will revert in case of insufficient balance
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L373
### Description
In the function below, the balance require check is not necessary due to `transferFrom()` implementation.
```solidity
    function reRoll(uint8 tokenId, uint8 fighterType) public {
        require(msg.sender == ownerOf(tokenId));
        require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
        require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll"); //Can delete

        _neuronInstance.approveSpender(msg.sender, rerollCost);
        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost); //Will revert if balance is insufficient
```
### Recommended Fix
Delete the `balanceOf()` require check.

## [G-14]<a name="g-14"></a> `FighterFarm._beforeTokenTransfer()` can be deleted to save an extra function call
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L451
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.9/contracts/token/ERC721/extensions/ERC721Enumerable.sol#L60
### Description
Note the code below:
```solidity
contract FighterFarm is ERC721, ERC721Enumerable {
    ...
    function _beforeTokenTransfer(address from, address to, uint256 tokenId)
        internal
        override(ERC721, ERC721Enumerable)
    {
        super._beforeTokenTransfer(from, to, tokenId);
    }
```
Inheritance in solidity is right-to-left and `super._beforeTokenTransfer(from, to, tokenId);` will call the `_beforeTokenTransfer()` method in `ERC721Enumerable`. Therefore, this function can be deleted and the functionality will be the same while eliminating an extra function call and saving gas.
### Recommended Fix
Remove `FighterFarm._beforeTokenTransfer()`.

## [G-15]<a name="g-15"></a> `FighterFarm._createFighterBase()` performs unnecessary check when minting dendroid
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L484-L531
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L462-L474
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L83-L94
### Description
Notice below that `_createNewFighter()` only uses `newDna` when calling `AiArenaHelper.createPhysicalAttributes()`:
```solidity
    function _createNewFighter(
        ...
        uint256 element; 
        uint256 weight;
        uint256 newDna;
        if (customAttributes[0] == 100) {
            (element, weight, newDna) = _createFighterBase(dna, fighterType);
        }
        else {
            element = customAttributes[0];
            weight = customAttributes[1];
            newDna = dna;
        }
        uint256 newId = fighters.length;

        bool dendroidBool = fighterType == 1;
        FighterOps.FighterPhysicalAttributes memory attrs = _aiArenaHelperInstance.createPhysicalAttributes(
            newDna,
            generation[fighterType],
            iconsType,
            dendroidBool
        );
        //newDna is not used after this point
    ...
    function _createFighterBase(
        uint256 dna, 
        uint8 fighterType
    ) 
        private 
        view 
        returns (uint256, uint256, uint256) 
    {
        uint256 element = dna % numElements[generation[fighterType]];
        uint256 weight = dna % 31 + 65;
        uint256 newDna = fighterType == 0 ? dna : uint256(fighterType); //This check is not needed
        return (element, weight, newDna);
    }
```
The above check in `_createFighterBase()` that sets dendroid fighters' (`fighterType` equals 1) `newDna` to a different value is not necessary, because `newDna` will not be used in `AiArenaHelper.createPhysicalAttributes()` when creating a dendroid and there are only two fighter types:
```solidity
    function createPhysicalAttributes(
        uint256 dna, 
        uint8 generation, 
        uint8 iconsType, 
        bool dendroidBool
    ) 
        external 
        view 
        returns (FighterOps.FighterPhysicalAttributes memory) 
    {
        if (dendroidBool) {
            return FighterOps.FighterPhysicalAttributes(99, 99, 99, 99, 99, 99);
```
### Recommended Fix
Instances of `newDna` can be removed and `dna` used instead, since `dna` will not change for non-dendroid fighters and `newDna` is not used when creating a dendroid fighter. 

## [G-16]<a name="g-16"></a> `MergingPool.claimRewards()` should update `numRoundsClaimed` after the loop ends instead of incrementing it in every loop
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L150
### Description
The below code is not gas efficient due to SSTORE/SLOAD of the `numRoundsClaimed` state variable in every loop.
```solidity
    function claimRewards(
        string[] calldata modelURIs, 
        string[] calldata modelTypes,
        uint256[2][] calldata customAttributes
    ) 
        external 
    {
        uint256 winnersLength;
        uint32 claimIndex = 0;
        uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            numRoundsClaimed[msg.sender] += 1;
```
### Recommended Fix
Instead of incrementing the state variable in every loop, increment a local variable and then update the state variable after the loop ends.

## [G-17]<a name="g-17"></a> `MergingPool.pickWinner()` and `MergingPool.claimRewards()` can be refactored so that users don't have to iterate through the entire array of winners for every round
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L151-L153
### Description
Notice how `claimRewards()` detects if the caller has rewards to claim in a round:
```solidity
            winnersLength = winnerAddresses[currentRound].length;
            for (uint32 j = 0; j < winnersLength; j++) {
                if (msg.sender == winnerAddresses[currentRound][j]) {
```
This method is very gas inefficient since `winnerAddresses` is in storage.
### Recommended Fix
Instead of iterating through all the winner addresses in every round, `MergingPool.pickWinner()` and `MergingPool.claimRewards()` could be refactored to increment a new state variable tracking number of rewards (per round, if desired).

## [G-18]<a name="g-18"></a> DNA rarity calculation in `AiArenaHelper.dnaToIndex()` can be redesigned to save gas
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L108
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L169-L186
### Description
`dnaToIndex()` is called inside a loop for all the fighter physical properties (see first link), which consumes a lot of gas due to storage loads. The system of calculating `attributeProbabilityIndex` by looping through the probabilities and comparing the `cumProb` to `rarityRank` could be refactored to retain the same functionality but save a lot of gas.

`dnaToIndex()` function below for reader convenience:
```solidity
    function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute) 
        public 
        view 
        returns (uint256 attributeProbabilityIndex) 
    {
        uint8[] memory attrProbabilities = getAttributeProbabilities(generation, attribute);
        
        uint256 cumProb = 0;
        uint256 attrProbabilitiesLength = attrProbabilities.length;
        for (uint8 i = 0; i < attrProbabilitiesLength; i++) {
            cumProb += attrProbabilities[i];
            if (cumProb >= rarityRank) {
                attributeProbabilityIndex = i + 1;
                break;
            }
        }
        return attributeProbabilityIndex;
    }
```
### Recommended Fix
Refactor attribute index/rarity calculation.

## [G-19]<a name="g-19"></a> `AiArenaHelper.createPhysicalAttributes()` should check for `iconsType` before running icons-relevant code since icons fighters are rare
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L100-L105
### Description
Since icon fighter NFTs are rare, the below checks related to icons should not run for non-icon fighters.
```solidity
            for (uint8 i = 0; i < attributesLength; i++) {
                if (
                  i == 0 && iconsType == 2 || // Custom icons head (beta helmet)
                  i == 1 && iconsType > 0 || // Custom icons eyes (red diamond)
                  i == 4 && iconsType == 3 // Custom icons hands (bowling ball)
                ) {
                    finalAttributeProbabilityIndexes[i] = 50;
                } else {
                    //set finalAttributeProbabilityIndexes[i] a different way
```
### Recommended Fix
Instead of running through all 3 icons checks every time, check to see if `iconsType` is greater than zero before running through the 3 checks.

## [G-20]<a name="g-20"></a> `AiArenaHelper` constructor sets probabilities twice
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L41-L52
### Description
Notice below that the constructor sets the probabilities twice:
```solidity
    constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 
```
### Recommended Fix
Remove the `for` loop.