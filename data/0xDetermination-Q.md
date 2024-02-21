# QA report by 0xDetermination

|Issue number|Description|
|:-|:-|
|Low issues|
|[L-1](#l-1)|Discrepancy between docs and code- false `initiatorBool` argument for `RankedBattle.updateBattleRecord()` should not change a fighter's points according to docs |
|[L-2](#l-2)|Small stake amounts can't be lost to `StakeAtRisk` due to precision loss|
|[L-3](#l-3)|`RankedBattle.claimNRN()` provides extra attack surface in case of an attacker inflating `claimableNRN` value|
|[L-4](#l-4)|New users to the protocol joining a significant amount of time after project launch will incur huge gas fees when calling `RankedBattle.claimNRN()`|
|[L-5](#l-5)|`FighterFarm.IncrementGeneration()` should enforce a `fighterType` of 0 or 1|
|[L-6](#l-6)|`MergingPool.claimRewards()` can mint an NFT with out-of-range fighter weight|
|[L-7](#l-7)||
|Info issues|
|[I-1](#i-1)|Comments for win/lose case can be put on the same line as the conditional statement for clarity|
|[I-2](#i-2)|Consider calling `GameItems.instantiateNeuronContract()` in the constructor|
|[I-3](#i-3)|`success` bool checks on NRN transfers are not necessary because the OZ implementation will not silently fail|
|[I-4](#i-4)|Explicitly declare state variable visibility for code clarity|
|[I-5](#i-5)|Ambiguous NatSpec for `MergingPool.claimRewards()`|
|[I-6](#i-6)|Consider adding a `minId` param to `MergingPool.getFighterPoints()`|
|[I-7](#i-7)|`Neuron.burnFrom()` is not used in the protocol and can be removed to reduce attack surface area|
|[I-8](#i-8)|`FighterFarm.reRoll()` NatSpec should specify pseudo-randomness|
|[I-9](#i-9)|Fighter generations can't increase past 255; `FighterFarm.incrementGeneration()` will fail with overflow|
|[I-10](#i-10)|Consider using interfaces instead of importing contracts directly|
|[I-11](#i-11)|`FighterFarm.sol` doesn't need to inherit `ERC721` since it already inherits `ERC721Enumerable`|
|[I-12](#i-12)|Fighter NFT properties may change unexpectedly after being transferred to a buyer (for example, if the fighter entered a match just before the sale)|

## [L-1]<a name="l-1"></a> Discrepancy between docs and code- false `initiatorBool` argument for `RankedBattle.updateBattleRecord()` should not change a fighter's points according to docs 
### Links
https://docs.aiarena.io/gaming-competition/nrn-rewards#0e10abc361f64aebae0ed48432213724
### Description
From the docs linked above: 'Only the Challenger NFT accrues Points during a match. For the Opponent NFT, the outcome does not impact their Point total.' However, the code in `RankedBattle.updateBattleRecord()` only uses the `initiatorBool` to check whether or not the challenger has sufficient voltage. (Link to the function: https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L322-L349)

Therefore, a non-challenger will have its points updated.
### Recommended Fix
Refactor the code to reflect the docs, or update the docs.

## [L-2]<a name="l-2"></a> Small stake amounts can't be lost to `StakeAtRisk` due to precision loss
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L433-L446
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L519-L534
### Description
Notice how `RankedBattle.updateBattleRecord()` calculates the potential points earned and potential `stakeAtRisk` to take from the player:
```solidity
    function _addResultPoints(
        ...
        if (_calculatedStakingFactor[tokenId][roundId] == false) {
            stakingFactor[tokenId] = _getStakingFactor(tokenId, stakeAtRisk);
            _calculatedStakingFactor[tokenId][roundId] = true;
        }

        curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4; //Precision loss in stakeAtRisk calculation
        ...
            points = stakingFactor[tokenId] * eloFactor; //Points are calculated based on stakingFactor

    ...
    function _getStakingFactor(
        uint256 tokenId, 
        uint256 stakeAtRisk
    ) 
        private 
        view 
        returns (uint256) 
    {
      uint256 stakingFactor_ = FixedPointMathLib.sqrt(
          (amountStaked[tokenId] + stakeAtRisk) / 10**18
      );
      if (stakingFactor_ == 0) { //stakingFactor is set to 1 if it was calculated as zero
        stakingFactor_ = 1;
      }
      return stakingFactor_;
    }  
```
Because there is precision loss in the `curStakeAtRisk` calculation and the user's `stakingFactor` is set to 1 if it was calculated as 0, a user can have a zero `curStakeAtRisk` while earning a nonzero amount of points.

Users with small stake can essentially play risk-free and can earn more points than they should. However, because they won't win many points due to the very small stake, the protocol might desire to ignore this issue.
### Recommended Fix
Don't set the `stakingFactor` to 1 if it's calculated as 0.

## [L-3]<a name="l-3"></a> `RankedBattle.claimNRN()` provides extra attack surface in case of an attacker inflating `claimableNRN` value
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L294-L311
### Description
Notice below that the reward NRN is minted directly instead of being distributed from a reward pool:
```solidity
        if (claimableNRN > 0) {
            amountClaimed[msg.sender] += claimableNRN;
            _neuronInstance.mint(msg.sender, claimableNRN);
```
If an attacker is able to inflate the `claimableNRN` value somehow, an uncapped amount of NRN may be minted. This could destroy the protocol's tokenomics.
### Recommended Fix
Consider enforcing a reasonable max minting value, or transferring existing NRN instead of minting.

## [L-4]<a name="l-4"></a> New users to the protocol joining a significant amount of time after project launch will incur huge gas fees when calling `RankedBattle.claimNRN()`
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L294-L305
### Description
Notice in the below code that the for loop will run for every completed round:
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
Since there are at least 4 SLOADS in the loop and 1 SSTORE, the gas cost of each loop will be around or over 5000. If a user joins a significant amount of time after protocol launch, they could incur a huge gas cost and lose a lot of gas fees. For example, 200 rounds may pass after 2 years (the sponsor has stated that each round will be last for around half a week). In this case, the user may have to consume 1 million gas for the transaction.

At current average Arbitrum gas prices, this will cost around 14 USDC.
### Recommended Fix
Allow users to set the `lowerBound` for the loop to an arbitrary round and update their `numRoundsClaimed` to that round.

## [L-5]<a name="l-5"></a> `FighterFarm.IncrementGeneration()` should enforce a `fighterType` of 0 or 1
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L125-L134
### Description
There are only two fighter types (0 or 1), so it should not be possible to call `incrementGeneration()` with an arbitrary `uint8`. This may result in an increased attack surface.
```solidity
    /// @param fighterType Type of fighter either 0 or 1 (champion or dendroid).
    /// @return Generation count of the fighter type.
    function incrementGeneration(uint8 fighterType) external returns (uint8) {
        require(msg.sender == _ownerAddress);
        generation[fighterType] += 1;
        maxRerollsAllowed[fighterType] += 1;
        return generation[fighterType];
    }
```
### Recommended Fix
Enforce the fighter type, either with a `require` statement or changing the parameter type to `bool`.

## [L-6]<a name="l-6"></a> `MergingPool.claimRewards()` can mint an NFT with out-of-range fighter weight
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L139-L159
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L313-L331
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L484-L527
### Description
`MergingPool.claimRewards()` doesn't perform input validation on the `customAttributes` arg, allowing the user to mint an NFT with an out-of-range weight. (Weight range is [65-95] https://discord.com/channels/810916927919620096/1201981655850426399/1208908219754221568)

See call trace below:
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
            winnersLength = winnerAddresses[currentRound].length;
            for (uint32 j = 0; j < winnersLength; j++) {
                if (msg.sender == winnerAddresses[currentRound][j]) {
                    _fighterFarmInstance.mintFromMergingPool( //next call
                        msg.sender,
                        modelURIs[claimIndex],
                        modelTypes[claimIndex],
                        customAttributes[claimIndex]
                    );
```
```solidity
    function mintFromMergingPool(
        address to, 
        string calldata modelHash, 
        string calldata modelType, 
        uint256[2] calldata customAttributes
    ) 
        public 
    {
        require(msg.sender == _mergingPoolAddress);
        _createNewFighter( //next call
            to, 
            uint256(keccak256(abi.encode(msg.sender, fighters.length))), 
            modelHash, 
            modelType,
            0,
            0,
            customAttributes
        );
    }
    ...
    function _createNewFighter(
        address to, 
        uint256 dna, 
        string memory modelHash,
        string memory modelType, 
        uint8 fighterType,
        uint8 iconsType,
        uint256[2] memory customAttributes
    ) 
        private 
    {  
        require(balanceOf(to) < MAX_FIGHTERS_ALLOWED);
        uint256 element; 
        uint256 weight;
        uint256 newDna;
        if (customAttributes[0] == 100) {
            (element, weight, newDna) = _createFighterBase(dna, fighterType);
        }
        else { //this block runs and sets arbitrary args
            element = customAttributes[0];
            weight = customAttributes[1];
            newDna = dna;
        }
        ...
        fighters.push(
            FighterOps.Fighter(
                weight,
                element,
                attrs,
                newId,
                modelHash,
                modelType,
                generation[fighterType],
                iconsType,
                dendroidBool
            )
        );
```
### Recommended Fix
Perform appropriate input validation in `MergingPool.claimRewards()`.

## [L-7]<a name="l-7"></a> Ensure off-chain validation is performed to check player voltage and prevent the game server from being gas griefed
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L322-L338
### Description
`RankedBattle.updateBattleRecord()` validates that a game initiator has enough voltage before updating the record. If this validation is only done on-chain, a malicious user may be able to spam initiate fights to gas grief the game server.
```solidity
    function updateBattleRecord(
        ...
        require(
            !initiatorBool ||
            _voltageManagerInstance.ownerVoltageReplenishTime(fighterOwner) <= block.timestamp || 
            _voltageManagerInstance.ownerVoltage(fighterOwner) >= VOLTAGE_COST
        );
```
### Recommended Fix
Ensure player voltage is checked off-chain before allowing them to start a battle.

## [I-1]<a name="i-1"></a> Comments for win/lose case can be put on the same line as the conditional statement for clarity
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L440-L441
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L472-L473
### Description
The comments for win/lose case of 0/2 in `RankedBattle._addResultPoints()` are below the conditional statements. These can be put inline with the conditional statements for clarity:
```solidity
        if (battleResult == 0) {
            /// If the user won the match
        ...
        } else if (battleResult == 2) {
            /// If the user lost the match
```
### Recommended Fix
Move the comments one line up.

## [I-2]<a name="i-2"></a> Consider calling `GameItems.instantiateNeuronContract()` in the constructor
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L95-L99
### Description
The constructor in `GameItems.sol` doesn't set the NRN contract:
```solidity
    constructor(address ownerAddress, address treasuryAddress_) ERC1155("https://ipfs.io/ipfs/") {
        _ownerAddress = ownerAddress;
        treasuryAddress = treasuryAddress_;
        isAdmin[_ownerAddress] = true;
    }
```
Consider setting the NRN contract in the constructor to avoid accidentally leaving it unset and the contract not working properly.
### Recommended Fix

## [I-3]<a name="i-3"></a> `success` bool checks on NRN transfers are not necessary because the OZ implementation will not silently fail
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L283-L284
### Description
There are many places in the codebase that follow the pattern:
```solidity
        bool success = _neuronInstance.transfer(msg.sender, amount);
        if (success) {
```
These checks are unnecessary, because the OZ implementation of ERC20 will always revert if the transfer has failed.
### Recommended Fix
Remove the transfer success caches/checks.

## [I-4]<a name="i-4"></a> Explicitly declare state variable visibility for code clarity
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L68-L75
### Description
Consider explicitly declaring the below state variables in `RankedBattle.sol` as `internal`.
```solidity
    /// The StakeAtRisk contract address.
    address _stakeAtRiskAddress;

    /// The address that has owner privileges (initially the contract deployer).
    address _ownerAddress;

    /// @notice The address in charge of updating battle records.
    address _gameServerAddress;
```
### Recommended Fix
See description.

## [I-5]<a name="i-5"></a> Ambiguous NatSpec for `MergingPool.claimRewards()`
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L135-L139
### Description
The NatSpec below is somewhat confusing/ambiguous. 
```solidity
    /// @dev The user can only claim rewards once for each round.
    /// @param modelURIs The array of model URIs corresponding to each round and winner address.
    /// @param modelTypes The array of model types corresponding to each round and winner address.
    /// @param customAttributes Array with [element, weight] of the newly created fighter.
    function claimRewards(
```
'The user can only claim rewards once for each round' may imply that users can only claim one NFT per round, but that is not true since the `pickWinner()` function can pick the same address as a winner multiple times in a round (https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L125).

Likewise, the phrasing of 'The array of model URIs corresponding to each round and winner address' somewhat implies that users can only claim one NFT per round. Also, 'corresponding to each round and winner address' implies that the arguments may include NFT data for more than one user, but this is not the case. (Each user can only claim for themselves.)
### Recommended Fix
Refactor the NatSpec.

## [I-6]<a name="i-6"></a>  Consider adding a `minId` param to `MergingPool.getFighterPoints()`
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L205-L211
### Description
`MergingPool.getFighterPoints()` allows the caller to specify a max token ID up to which fighter points will be retrieved. Consider adding a `minId` as well to improve readability since there will be many tokens. This will not introduce extra attack surface area since the function is not used anywhere in the code.
```solidity
    function getFighterPoints(uint256 maxId) public view returns(uint256[] memory) {
        uint256[] memory points = new uint256[](1);
        for (uint256 i = 0; i < maxId; i++) {
            points[i] = fighterPoints[i];
        }
        return points;
    }
``` 
### Recommended Fix
Set `i` to a user-provided `minId`.

## [I-7]<a name="i-7"></a> `Neuron.burnFrom()` is not used in the protocol and can be removed to reduce attack surface area
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L196
### Description
Since the `burnFrom()` function is not used anywhere in the code, it can be deleted to reduce attack surface area.
### Recommended Fix
Delete `burnFrom()`.

## [I-8]<a name="i-8"></a> `FighterFarm.reRoll()` NatSpec should specify pseudo-randomness
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L367
### Description
The NatSpec 'Rolls a new fighter with random traits' is somewhat misleading; trait determination is pseudorandom.
### Recommended Fix
Change the comment to 'Rerolls a fighter with pseudorandom traits.'

## [I-9]<a name="i-9"></a> Fighter generations can't increase past 255; `FighterFarm.incrementGeneration()` will fail with overflow
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L36
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L129-L134
### Description
Notice below that due to the type of `maxRerollsAllowed`, fighter generation can't be incremented past 255:
```solidity
    uint8[2] public maxRerollsAllowed = [3, 3];
    ...
    function incrementGeneration(uint8 fighterType) external returns (uint8) {
        require(msg.sender == _ownerAddress);
        generation[fighterType] += 1;
        maxRerollsAllowed[fighterType] += 1;
        return generation[fighterType];
    }
```
### Recommended Fix
The type of `maxRerollsAllowed` and related variables can be changed to a larger uint type if the protocol anticipates incrementing fighter generations past 255.

## [I-10]<a name="i-10"></a> Consider using interfaces instead of importing contracts directly
### Links
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L4-L10
### Description
The contracts in the codebase follow a pattern of importing contracts directly instead of using interfaces. (The common practice is to use interfaces, although this seems to be a style choice.)
### Recommended Fix
Use interfaces if the protocol desires that style choice.

## [I-11]<a name="i-11"></a> `FighterFarm.sol` doesn't need to inherit `ERC721` since it already inherits `ERC721Enumerable`
### Links
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.9/contracts/token/ERC721/extensions/ERC721Enumerable.sol#L14
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L16
### Description
Solidity inheritance is ordered right-to-left and `ERC721Enumerable` already inherits `ERC721`. The overrides and super calls will be unchanged if the contract only inherits `ERC721Enumerable`.

```solidity
contract FighterFarm is ERC721, ERC721Enumerable {
```
### Recommended Fix
Only inheriting `ERC721Enumerable`.

## [I-12]<a name="i-12"></a> Fighter NFT properties may change unexpectedly after being transferred to a buyer (for example, if the fighter entered a match just before the sale)
### Links
n/a
### Description
Since fighters are always available for battles, the NFT properties (such as win stats) could change unexpectedly for a buyer. This can happen if the fighter entered a match just before the sale. Note that whether or not the fighter is in a battle is not recorded on-chain.
### Recommended Fix
There should be an offchain mechanism for buyers to see whether or not an NFT is engaged in battle. It may not be practical to attempt to prevent this issue on-chain since fighters should always be available for battles.