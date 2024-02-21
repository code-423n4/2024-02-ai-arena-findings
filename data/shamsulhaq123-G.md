## Gas Optimisition

## Summary

|No|Issue|Instance|
|--|-----|--------|
|[G-01]|+= costs more gas than = + for state variables|22|
|[G-02]|Use assembly for loops|15|
|[G-03]|Use shift Left instead of multiplication if possible to save gas|6|
|[G-04]|Use shift Right instead of division if possible to save gas|1|
|[G-05]|Using mappings instead of arrays to avoid length checks save gas|8|
|[G-06]|Should use arguments instead of state variable |17|
|[G-07]| Before transfer of  some functions, we should check some variables for possible gas save|6|
|[G-08]| Unnecessary look up in if condition|1|
|[G-09]|Use hardcode address instead address(this)|1|
|[G-10]|Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)|19|
|[G-11]| Optimize Address Storage Value Management with assembly|10|
|[G-12]|Use assembly to emit events|15|
|[G-13]|Cache array length outside of loop|12|
|[G-14]|Divisions which do not divide by -X cannot overflow or overflow so such operations can be unchecked to save gas|1|
|[G-15]| Do not calculate constants|3|
|[G-16]| Stack variable cost less while used in emiting event|17|
|[G-17]| Superfluous event fields|7|
|[G-18]|Consider merging sequential for loops|12|
|[G-19]|Not using the named return variables anywhere in the function is confusing|28|

## [G-01] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: AiArenaHelper.sol

179  cumProb += attrProbabilities[i];
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L179

```solidity

file: FighterFarm.sol

131   generation[fighterType] += 1;
132   maxRerollsAllowed[fighterType] += 1;

209 nftsClaimed[msg.sender][0] += numToMint[0];
210  nftsClaimed[msg.sender][1] += numToMint[1];

293  numTrained[tokenId] += 1;
294   totalNumTrained += 1;

378  numRerolls[tokenId] += 1;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L131-L132
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L209-L210
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L293-L294
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L378

```solidity

file: GameItems.sol

234 _itemCount += 1;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L234

```solidity

file: MergingPool.sol

131  roundId += 1;

150  numRoundsClaimed[msg.sender] += 1;

160 claimIndex += 1;

180  numRewards += 1;

197 fighterPoints[tokenId] += points;
198   totalPoints += points;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L131
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L150
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L160
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L180
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L197-L198

```solidity

file: RankedBattle.sol

236   roundId += 1;

256 amountStaked[tokenId] += amount;
257    globalStakedAmount += amount;

301 claimableNRN += (

304   numRoundsClaimed[msg.sender] += 1;

307  amountClaimed[msg.sender] += claimableNRN;

348   totalBattles += 1;

392   claimableNRN += (

462   amountStaked[tokenId] += curStakeAtRisk;

466 accumulatedPointsPerFighter[tokenId][roundId] += points;
            accumulatedPointsPerAddress[fighterOwner][roundId] += points;
468           totalAccumulatedPoints[roundId] += point

507     fighterBattleRecord[tokenId].wins += 1;
        } else if (battleResult == 1) {
            fighterBattleRecord[tokenId].ties += 1;
        } else if (battleResult == 2) {
511           fighterBattleRecord[tokenId].loses += 1;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L236
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L256-L257
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L301
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L304
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L307
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L348
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L392
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L462
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L466-L468
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L507-L511

```solidity

file: StakeAtRisk.sol

123 stakeAtRisk[roundId][fighterId] += nrnToPlaceAtRisk;
        totalStakeAtRisk[roundId] += nrnToPlaceAtRisk;
125    amountLost[fighterOwner] += nrnToPlaceAtRisk;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L123-L125 

## [G-02]  Use assembly for loops
In the following instances, assembly is used for more gas efficient loops.

```solidity

file: AiArenaHelper.sol

48   for (uint8 i = 0; i < attributesLength; i++) 

73 for (uint8 i = 0; i < attributesLength; i++) 

99   for (uint8 i = 0; i < attributesLength; i++) 

136 for (uint8 i = 0; i < attributesLength; i++) 

148  for (uint8 i = 0; i < attributesLength; i++)
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L48
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L73
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L99
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L136
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L148
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L178

```solidity

file: FighterFarm.sol

211   for (uint16 i = 0; i < totalToMint; i++)

249   for (uint16 i = 0; i < mintpassIdsToBurn.length; i++)
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L211
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L249

```solidity

file: MergingPool.sol

124  for (uint256 i = 0; i < winnersLength; i++)

149    for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            numRoundsClaimed[msg.sender] += 1;
            winnersLength = winnerAddresses[currentRound].length;
152         for (uint32 j = 0; j < winnersLength; j++)

176  for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            winnersLength = winnerAddresses[currentRound].length;
178    for (uint32 j = 0; j < winnersLength; j++)

207 for (uint256 i = 0; i < maxId; i++)
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L124
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L149-L152
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L176-L178
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L207

```solidity

file: Neuron.sol

131  for (uint32 i = 0; i < recipientsLength; i++)
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L131

```solidity

file: RankedBattle.sol

299   for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) 

390  for (uint32 i = lowerBound; i < roundId; i++) 
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L299
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L390

## [G-03] Use shift Left instead of multiplication if possible to save gas

```solidity

file: RankedBattle.sol

157  rankedNrnDistribution[0] = 5000 * 10**18;

220  rankedNrnDistribution[roundId] = newDistribution * 10**18;

439  curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;

445    points = stakingFactor[tokenId] * eloFactor;

449  uint256 mergingPoints = (points * mergingPortion) / 100

481   points = stakingFactor[tokenId] * eloFactor;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L157
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L220
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L439
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L445
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L449
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L481

## [G-04] Use shift Right instead of division if possible to save gas

```solidity

file: RankedBattle.sol

449  uint256 mergingPoints = (points * mergingPortion) / 100;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L449

## [G-05] Using mappings instead of arrays to avoid length checks save gas
Just by using a mapping, we get a gas saving of 2102 gas. When you read the value of an index of an array, solidity adds bytecode that checks that you are reading from a valid index 

```solidity

file: AiArenaHelper.sol

17  string[] public attributes = ["head", "eyes", "mouth", "body", "hands", "feet"];

    /// @notice Default DNA divisors for attributes
20    uint8[] public defaultAttributeDivisor = [2, 3, 5, 7, 11, 13];

41  constructor(uint8[][] memory probabilities) 

68   function addAttributeDivisor(uint8[] memory attributeDivisors) external 

96   uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributes.length);

131    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public 

149   attributeProbabilities[generation][attributes[i]] = new uint8[](0);

174  uint8[] memory attrProbabilities = getAttributeProbabilities(generation, attribute);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L17
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L41
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L68
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L96
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L149
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L174

```soidity

file: FighterFarm.sol

36   uint8[2] public maxRerollsAllowed = [3, 3];

    /// @notice The cost ($NRN) to reroll a fighter.
    uint256 public rerollCost = 1000 * 10**18;    

    /// @notice Stores the current generation for each fighter type.
42    uint8[2] public generation = [0, 0];
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L36-L42 

## [G-06] Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas   

```solidity

file: FighterFarm.sol

272   emit Locked(tokenId);
        } else {
274       emit Unlocked(tokenId);

61   emit FighterCreated(id, weight, element, generation);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L272-L274
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L61

```solidity

file: GameItems.sol

130    emit Unlocked(tokenId);
        } else {
132      emit Locked(tokenId);

174 emit BoughtItem(msg.sender, tokenId, quantity);

231   emit Locked(_itemCount);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L130-L132
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L174
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L231

```solidity

file: MergingPool.sol

165  emit Claimed(msg.sender, claimIndex);

199  emit PointsAdded(tokenId, points);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L165
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L199

```solidity

file: Neuron.sol

144  emit TokensClaimed(msg.sender, amount);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L144

```solidity

file: RankedBattle.sol

263  emit Staked(msg.sender, amount);

288  emit Unstaked(msg.sender, amount);

309  emit Claimed(msg.sender, claimableNRN);

470  emit PointsChanged(tokenId, points, true);

489  emit PointsChanged(tokenId, points, false);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L263
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L288
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L309
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L470
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L489

```solidity

file: StakeAtRisk.sol

105  emit ReclaimedStake(fighterId, nrnToReclaim);

126  emit IncreasedStakeAtRisk(fighterId, nrnToPlaceAtRisk);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L105
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L126

```solidity

file: VoltageManager.sol

98  emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);

111    emit VoltageRemaining(spender, ownerVoltage[spender]);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L98
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L111

## [G-07] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:

``solidity

file: GameItems.sol

242  function burn(address account, uint256 tokenId, uint256 amount) public {
        require(allowedBurningAddresses[msg.sender]);
        _burn(account, tokenId, amount);
    }

291  function safeTransferFrom(
        address from, 
        address to, 
        uint256 tokenId,
        uint256 amount,
        bytes memory data
    )   

302    super.safeTransferFrom(from, to, tokenId, amount, data);      
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L242
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L291
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L302

```solidity

file: MergingPool.sol

106  function updateWinnersPerPeriod(uint256 newWinnersPerPeriodAmount) external {
        require(isAdmin[msg.sender]);
        winnersPerPeriod = newWinnersPerPeriodAmount;
    }   
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L106

```solidity

file: Neuron.sol

127     function setupAirdrop(address[] calldata recipients, uint256[] calldata amounts) external {
        require(isAdmin[msg.sender]);
        require(recipients.length == amounts.length);
        uint256 recipientsLength = recipients.length;
        for (uint32 i = 0; i < recipientsLength; i++) {
            _approve(treasuryAddress, recipients[i], amounts[i]);
        }
    }

    /// @notice Claims the specified amount of tokens from the treasury address to the caller's address.
    /// @param amount The amount to be claimed
    function claim(uint256 amount) external {
        require(
            allowance(treasuryAddress, msg.sender) >= amount, 
            "ERC20: claim amount exceeds allowance"
        );
        transferFrom(treasuryAddress, msg.sender, amount);
        emit TokensClaimed(msg.sender, amount);
    }

    /*//////////////////////////////////////////////////////////////
                            PUBLIC FUNCTIONS
    //////////////////////////////////////////////////////////////*/

    /// @notice Mints the specified amount of tokens to the given address.
    /// @dev The caller must have the minter role.
    /// @param to The address to which the tokens will be minted.
    /// @param amount The amount of tokens to be minted.
    function mint(address to, uint256 amount) public virtual {
        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }

    /// @notice Burns the specified amount of tokens from the caller's address.
    /// @param amount The amount of tokens to be burned.
    function burn(uint256 amount) public virtual {
        _burn(msg.sender, amount);
    }

    /// @notice Approves the specified amount of tokens for the spender address.
    /// @dev The caller must have the spender role.
    /// @param account The account for which to approve the allowance.
    /// @param amount The amount of tokens to be approved.
    function approveSpender(address account, uint256 amount) public {
        require(
            hasRole(SPENDER_ROLE, msg.sender), 
            "ERC20: must have spender role to approve spending"
        );
        _approve(account, msg.sender, amount);
    }

    /// @notice Approves the specified amount of tokens for the staker address.
    /// @dev The caller must have the staker role.
    /// @param owner The owner of the tokens.
    /// @param spender The address for which to approve the allowance.
    /// @param amount The amount of tokens to be approved.
    function approveStaker(address owner, address spender, uint256 amount) public {
        require(
            hasRole(STAKER_ROLE, msg.sender), 
            "ERC20: must have staker role to approve staking"
        );
        _approve(owner, spender, amount);
    }

    /// @notice Burns the specified amount of tokens from the account address.
    /// The caller must have an allowance greater than or equal to the amount.
    /// @param account The account from which to burn the tokens
    /// @param amount The amount of tokens to be burned
    function burnFrom(address account, uint256 amount) public virtual {
        require(
            allowance(account, msg.sender) >= amount, 
            "ERC20: burn amount exceeds allowance"
        );
        uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
        _burn(account, amount);
        _approve(account, msg.sender, decreasedAllowance);
    }
 
206 }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L127-L206

```solidity

file: RankedBattle.sol

244    function stakeNRN(uint256 amount, uint256 tokenId) external {
        require(amount > 0, "Amount cannot be 0");
        require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");
        require(_neuronInstance.balanceOf(msg.sender) >= amount, "Stake amount exceeds balance");
        require(hasUnstaked[tokenId][roundId] == false, "Cannot add stake after unstaking this round");

        _neuronInstance.approveStaker(msg.sender, address(this), amount);
        bool success = _neuronInstance.transferFrom(msg.sender, address(this), amount);
        if (success) {
            if (amountStaked[tokenId] == 0) {
                _fighterFarmInstance.updateFighterStaking(tokenId, true);
            }
            amountStaked[tokenId] += amount;
            globalStakedAmount += amount;
            stakingFactor[tokenId] = _getStakingFactor(
                tokenId, 
                _stakeAtRiskInstance.getStakeAtRisk(tokenId)
            );
            _calculatedStakingFactor[tokenId][roundId] = true;
            emit Staked(msg.sender, amount);
        }
    }

    /// @notice Unstakes NRN tokens.
    /// @param amount The amount of NRN tokens to unstake.
    /// @param tokenId The ID of the token to unstake.
    function unstakeNRN(uint256 amount, uint256 tokenId) external {
        require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");
        if (amount > amountStaked[tokenId]) {
            amount = amountStaked[tokenId];
        }
        amountStaked[tokenId] -= amount;
        globalStakedAmount -= amount;
        stakingFactor[tokenId] = _getStakingFactor(
            tokenId, 
            _stakeAtRiskInstance.getStakeAtRisk(tokenId)
        );
        _calculatedStakingFactor[tokenId][roundId] = true;
        hasUnstaked[tokenId][roundId] = true;
        bool success = _neuronInstance.transfer(msg.sender, amount);
        if (success) {
            if (amountStaked[tokenId] == 0) {
                _fighterFarmInstance.updateFighterStaking(tokenId, false);
            }
            emit Unstaked(msg.sender, amount);
        }
290    }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L244-L290

## [G-08] Unnecessary look up in if condition

If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: AiArenaHelper.sol

100  if (
                  i == 0 && iconsType == 2 || // Custom icons head (beta helmet)
103                  i == 1 && iconsType > 0 || // Custom icons eyes (red diamond)
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L100-L103

## [G-09] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address.

```solidity

file: RankedBattle.sol

250 _neuronInstance.approveStaker(msg.sender, address(this), amount);
251   bool success = _neuronInstance.transferFrom(msg.sender, address(this), amount);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L250-251

## [G‑10] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

```solidity

file: AiArenaHelper.sol

42   _ownerAddress = msg.sender;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L42

```solidity

file: FighterFarm.sol

203   nftsClaimed[msg.sender][0],
       nftsClaimed[msg.sender][1]

209 nftsClaimed[msg.sender][0] += numToMint[0];
210   nftsClaimed[msg.sender][1] += numToMint[1];

212  _createNewFighter(
         msg.sender, 
214      uint256(keccak256(abi.encode(msg.sender, fighters.length))),

324   uint256(keccak256(abi.encode(msg.sender, fighters.length))), 

375 _neuronInstance.approveSpender(msg.sender, rerollCost);
        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);
        if (success) {
            numRerolls[tokenId] += 1;
379    uint256 dna = uint256(keccak256(abi.encode(msg.sender, tokenId, numRerolls[tokenId])));

541   _isApprovedOrOwner(msg.sender, tokenId) &&
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L203-204
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L209-210
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L212-L214
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L324
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L375-L379
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L541

```solidity

file: GameItems.sol

160  quantity <= allowanceRemaining[msg.sender][tokenId]

163     _neuronInstance.approveSpender(msg.sender, price);
        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, price);
        if (success) {
            if (dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp) {
                _replenishDailyAllowance(tokenId);
            }
            allowanceRemaining[msg.sender][tokenId] -= quantity;
            if (allGameItemAttributes[tokenId].finiteSupply) {
                allGameItemAttributes[tokenId].itemsRemaining -= quantity;
            }
            _mint(msg.sender, tokenId, quantity, bytes("random"));
174             emit BoughtItem(msg.sender, tokenId, quantity);

313  allowanceRemaining[msg.sender][tokenId] = allGameItemAttributes[tokenId].dailyAllowance;
314  dailyAllowanceReplenishTime[msg.sender][tokenId] = uint32(block.timestamp + 1 days);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L160
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L163-L174
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L313-L314

```solidity

file: MergingPool.sol

148    uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            numRoundsClaimed[msg.sender] += 1;
            winnersLength = winnerAddresses[currentRound].length;
            for (uint32 j = 0; j < winnersLength; j++) {
                if (msg.sender == winnerAddresses[currentRound][j]) {
                    _fighterFarmInstance.mintFromMergingPool(
                        msg.sender,
                        modelURIs[claimIndex],
                        modelTypes[claimIndex],
                        customAttributes[claimIndex]
159                   );

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L148-L159

```solidity

file: Neuron.sol

140        allowance(treasuryAddress, msg.sender) >= amount, 
            "ERC20: claim amount exceeds allowance"
        );
        transferFrom(treasuryAddress, msg.sender, amount);
        emit TokensClaimed(msg.sender, amount);
    }

164  _burn(msg.sender, amount);

173       hasRole(SPENDER_ROLE, msg.sender), 
            "ERC20: must have spender role to approve spending"
        );
        _approve(account, msg.sender, amount);
    }

186  hasRole(STAKER_ROLE, msg.sender), 

198      allowance(account, msg.sender) >= amount, 
            "ERC20: burn amount exceeds allowance"
        );
        uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
        _burn(account, amount);
203     _approve(account, msg.sender, decreasedAllowance);

250   _neuronInstance.approveStaker(msg.sender, address(this), amount);
251  bool success = _neuronInstance.transferFrom(msg.sender, address(this), amount);

295  require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
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
        if (claimableNRN > 0) {
            amountClaimed[msg.sender] += claimableNRN;
            _neuronInstance.mint(msg.sender, claimableNRN);
            emit Claimed(msg.sender, claimableNRN);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L140-L144
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L164
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L173-L176
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L186
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L198-L203
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L250-L251
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L295-L309

```solidity

file: VoltageManager.sol

94  require(ownerVoltage[msg.sender] < 100);
        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
        _gameItemsContractInstance.burn(msg.sender, 0, 1);
        ownerVoltage[msg.sender] = 100;
        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L94-L98

## [G-11] Optimize Address Storage Value Management with assembly

```solidity

file: AiArenaHelper.sol

42   _ownerAddress = msg.sender;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L42

```solidity

file: FighterFarm.sol

48     address public treasuryAddress;

    /// The address that has owner privileges (initially the contract deployer).
    address _ownerAddress;

    /// The address responsible for setting token URIs and signing fighter claim messages.
    address _delegatedAddress;

    /// The address of the Merging Pool contract.
    address _mergingPoolAddress;

    /// @dev Instance of the AI Arena Helper contract.
    AiArenaHelper _aiArenaHelperInstance;

    /// @dev Instance of the AI Arena Mintpass contract (ERC721).
    AAMintPass _mintpassInstance;

    /// @dev Instance of the Neuron contract (ERC20).
66    Neuron _neuronInstance;

107  _ownerAddress = ownerAddress;
        _delegatedAddress = delegatedAddress;
109    treasuryAddress = treasuryAddress_;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L48-L66
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L107-L109

```solidity

file: MergingPool.sol

91  _ownerAddress = newOwnerAddress;

108    winnersPerPeriod = newWinnersPerPeriodAmount;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L91
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L108

```solidity

file: Neuron.sol

71     _ownerAddress = ownerAddress;
72      treasuryAddress = treasuryAddress_;

87  _ownerAddress = newOwnerAddress;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L71-L72
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L87

```solidity

file: StakeAtRisk.sol

65  treasuryAddress = treasuryAddress_;
66    _rankedBattleAddress = rankedBattleAddress; 

82   roundId = roundId_;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L65-66
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L82

```solidity

file: VoltageManager.sol

66   _ownerAddress = newOwnerAddress;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L66

### [G-12] Use assembly to emit events
We can use assembly to emit events efficiently by utilizing `scratch space` and the `free memory pointer`. This will allow us to potentially avoid memory expansion costs. Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

```solidity

file: FighterFarm.sol

272     emit Locked(tokenId);
       
274     emit Unlocked(tokenId);

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L272

```solidity

file: FighterOps.sol

61  emit FighterCreated(id, weight, element, generation);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L61

```solidity

file: GameItems.sol

130    emit Unlocked(tokenId);
        
133      emit Locked(tokenId);

174  emit BoughtItem(msg.sender, tokenId, quantity);

231  emit Locked(_itemCount);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L130
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L174
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L231

```solidity

file: MergingPool.sol

165   emit Claimed(msg.sender, claimIndex);

199   emit PointsAdded(tokenId, points);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L165
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L199

```solidity

file: Neuron.sol

144    emit TokensClaimed(msg.sender, amount);

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L144

```solidity

file: RankedBattle.sol

263   emit Staked(msg.sender, amoun

288  emit Unstaked(msg.sender, amount);

309   emit Claimed(msg.sender, claimableNRN);

470   emit PointsChanged(tokenId, points, true);

489  emit PointsChanged(tokenId, points, false);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L263
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L288
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L309
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L470
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L489

```solidity

file: StakeAtRisk.sol

105   emit ReclaimedStake(fighterId, nrnToReclaim);

126  emit IncreasedStakeAtRisk(fighterId, nrnToPlaceAtRisk);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L105
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L126

## [G-13] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

```solidity

file: AiArenaHelper.sol

48  for (uint8 i = 0; i < attributesLength; i++) 

73   for (uint8 i = 0; i < attributesLength; i++)

99    for (uint8 i = 0; i < attributesLength; i++) 

136    for (uint8 i = 0; i < attributesLength; i++)

148  for (uint8 i = 0; i < attributesLength; i++)

178  for (uint8 i = 0; i < attrProbabilitiesLength; i++) 
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L48
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L73
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L99
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L136
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L148
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L178

```solidity

file: FighterFarm.sol

249  for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) 

211  for (uint16 i = 0; i < totalToMint; i++)
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L211
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L249

```solidity

file: MergingPool.sol

124  for (uint256 i = 0; i < winnersLength; i++) 

152 for (uint32 j = 0; j < winnersLength; j++)

178  for (uint32 j = 0; j < winnersLength; j++) 

207   for (uint256 i = 0; i < maxId; i++) 
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L152
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L178
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L207

```solidity

file: Neuron.sol

131  for (uint32 i = 0; i < recipientsLength; i++) 

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L131

## [G-14] Divisions which do not divide by -X cannot overflow or overflow so such operations can be unchecked to save gas
Make such found divisions are unchecked when ensured it is safe to do so

```solidity

file: AiArenaHelper.sol

107  uint256 rarityRank = (dna / attributeToDnaDivisor[attributes[i]]) % 100;


```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L107

## [G-15]  Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```solidity

file: FighterFarm.sol

39  uint256 public rerollCost = 1000 * 10**18;  


```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L39

```solidity

file: Neuron.sol

37     uint256 public constant INITIAL_TREASURY_MINT = 10**18 * 10**8 * 2;

    /// @notice Initial supply of NRN tokens to be minted and distributed to contributors.
    uint256 public constant INITIAL_CONTRIBUTOR_MINT = 10**18 * 10**8 * 5;

    /// @notice Maximum supply of NRN tokens.
43  uint256 public constant MAX_SUPPLY = 10**18 * 10**9;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L37-L43

```solidity

file: RankedBattle.sol

439 curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L439

## [G-16] Stack variable cost less while used in emiting event
Even if the variable is going to be used only one time, caching a state variable and use its cache in an emit would help you reduce the cost by at least

```solidity

file: FighterFarm.sol

272   emit Locked(tokenId);
        } else {
274       emit Unlocked(tokenId);

61   emit FighterCreated(id, weight, element, generation);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L272-L274
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L61

```solidity

file: GameItems.sol

130    emit Unlocked(tokenId);
        } else {
132      emit Locked(tokenId);

174 emit BoughtItem(msg.sender, tokenId, quantity);

231   emit Locked(_itemCount);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L130-L132
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L174
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L231

```solidity

file: MergingPool.sol

165  emit Claimed(msg.sender, claimIndex);

199  emit PointsAdded(tokenId, points);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L165
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L199

```solidity

file: Neuron.sol

144  emit TokensClaimed(msg.sender, amount);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L144

```solidity

file: RankedBattle.sol

263  emit Staked(msg.sender, amount);

288  emit Unstaked(msg.sender, amount);

309  emit Claimed(msg.sender, claimableNRN);

470  emit PointsChanged(tokenId, points, true);

489  emit PointsChanged(tokenId, points, false);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L263
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L288
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L309
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L470
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L489

```solidity

file: StakeAtRisk.sol

105  emit ReclaimedStake(fighterId, nrnToReclaim);

126  emit IncreasedStakeAtRisk(fighterId, nrnToPlaceAtRisk);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L105
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L126

```solidity

file: VoltageManager.sol

98  emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);

111    emit VoltageRemaining(spender, ownerVoltage[spender]);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L98
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L111

## [G-17] Superfluous event fields
`block.timestamp` and `block.number` are added to event information by default so adding them manually wastes gas

```solidity

file: GameItems.sol

59  dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp || 


166  if (dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp)

270  if (dailyAllowanceReplenishTime[owner][tokenId] <= block.timestamp) 

314   dailyAllowanceReplenishTime[msg.sender][tokenId] = uint32(block.timestamp + 1 days);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L159
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L166
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L270
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L314

```solidity

file: RankedBattle.sol

336  _voltageManagerInstance.ownerVoltageReplenishTime(fighterOwner) <= block.timestamp || 

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L336


```solidity

file: VoltageManager.sol

107 if (ownerVoltageReplenishTime[spender] <= block.timestamp) 


119  ownerVoltageReplenishTime[owner] = uint32(block.timestamp + 1 days);
      
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L107
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L119

## [G-18] Consider merging sequential for loops
Merging multiple `for` loops within a function in Solidity can enhance efficiency and reduce gas costs, especially when they share a common iterating variable or perform related operations. By minimizing redundant iterations over the same data set, execution becomes more cost-effective. However, while merging can optimize gas usage and simplify logic, it may also increase code complexity. Therefore, careful balance between optimization and maintainability is essential, along with thorough testing to ensure the refactored code behaves as expected.

```solidity

file: AiArenaHelper.sol

48  for (uint8 i = 0; i < attributesLength; i++) 

73   for (uint8 i = 0; i < attributesLength; i++)

99    for (uint8 i = 0; i < attributesLength; i++) 

136    for (uint8 i = 0; i < attributesLength; i++)

148  for (uint8 i = 0; i < attributesLength; i++)

178  for (uint8 i = 0; i < attrProbabilitiesLength; i++) 
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L48
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L73
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L99
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L136
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L148
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L178

```solidity

file: FighterFarm.sol

249  for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) 

211  for (uint16 i = 0; i < totalToMint; i++)
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L211
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L249

```solidity

file: MergingPool.sol

124  for (uint256 i = 0; i < winnersLength; i++) 

152 for (uint32 j = 0; j < winnersLength; j++)

178  for (uint32 j = 0; j < winnersLength; j++) 

207   for (uint256 i = 0; i < maxId; i++) 
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L152
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L178
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L207

```solidity

file: Neuron.sol

131  for (uint32 i = 0; i < recipientsLength; i++) 

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L131

## [G-19] Not using the named return variables anywhere in the function is confusing
Consider changing the variable to be an unnamed one, since the variable is never assigned, nor is it returned by name. If the optimizer is not turned on, leaving the code as it is will also waste gas for the stack variable.

```solidity

file: AiArenaHelper.sol

61  function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
    }

68   function addAttributeDivisor(uint8[] memory attributeDivisors) external {
        require(msg.sender == _ownerAddress);
        require(attributeDivisors.length == attributes.length);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeToDnaDivisor[attributes[i]] = attributeDivisors[i];
        }

131    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public

144   function deleteAttributeProbabilities(uint8 generation) public {
        require(msg.sender == _ownerAddress);

157       public 
        view 
        returns (uint8[] memory) 
    {
        return attributeProbabilities[generation][attribute];
    } 

169   function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute) 
        public 
        view 
        returns (uint256 attributeProbabilityIndex)     
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L61
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L68
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L144
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L157
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L169

```solidity

file: FighterFarm.sol

120  function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
    }

129   function incrementGeneration(uint8 fighterType) external returns (uint8) {
        require(msg.sender == _ownerAddress);
        generation[fighterType] += 1;
        maxRerollsAllowed[fighterType] += 1;
        return generation[fighterType];
    }    

139 function addStaker(address newStaker) external {
        require(msg.sender == _ownerAddress);
        hasStakerRole[newStaker] = true;
    }

147  function instantiateAIArenaHelperContract(address aiArenaHelperAddress) external {
        require(msg.sender == _ownerAddress);
        _aiArenaHelperInstance = AiArenaHelper(aiArenaHelperAddress);
    }    

155   function instantiateMintpassContract(address mintpassAddress) external {
        require(msg.sender == _ownerAddress);
        _mintpassInstance = AAMintPass(mintpassAddress);
    }

163  function instantiateNeuronContract(address neuronAddress) external {
        require(msg.sender == _ownerAddress);
        _neuronInstance = Neuron(neuronAddress);
    }    

171   function setMergingPoolAddress(address mergingPoolAddress) external {
        require(msg.sender == _ownerAddress);
        _mergingPoolAddress = mergingPoolAddress;
    }    

180     

268   function updateFighterStaking(uint256 tokenId, bool stakingStatus) external {
        require(hasStakerRole[msg.sender]);
        fighterStaked[tokenId] = stakingStatus;
        if (stakingStatus) {
            emit Locked(tokenId);

299  function doesTokenExist(uint256 tokenId) external view returns (bool) {
        return _exists(tokenId);
    }            

370  function reRoll(uint8 tokenId, uint8 fighterType) public {
        require(msg.sender == ownerOf(tokenId)); 

402 function tokenURI(uint256 tokenId) public view override(ERC721) returns (string memory) {
        return _tokenURIs[tokenId];
    }           

410  function supportsInterface(bytes4 _interfaceId)
        public
        view
        override(ERC721, ERC721Enumerable)
        returns (bool)    

447    function _beforeTokenTransfer(address from, address to, uint256 tokenId)
        internal
        override(ERC721, ERC721Enumerable)
    {        

462   function _createFighterBase(
        uint256 dna, 
        uint8 fighterType
    ) 
        private 
        view 
        returns (uint256, uint256, uint256)     

539  /// @return Bool whether the transfer is allowed or not.
    function _ableToTransfer(uint256 tokenId, address to) private view returns(bool) {
        return (
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L120
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L129
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L139
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L147
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L155
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L163
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L171
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L180
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L268
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L299
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L370
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L402
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L410
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L447
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L462
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L539

```solidity

file: FighterOps.sol

67   function getFighterAttributes(Fighter storage self) public view returns (uint256[6] memory)

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L67

```solidity

file: GameItems.sol

108  function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
    }

117  function adjustAdminAccess(address adminAddress, bool access) external {
        require(msg.sender == _ownerAddress);
        isAdmin[adminAddress] = access;
    }      

126  function adjustTransferability(uint256 tokenId, bool transferable) external {
        require(msg.sender == _ownerAddress);
        allGameItemAttributes[tokenId].transferable = transferable;  

139  function instantiateNeuronContract(address nrnAddress) external {
        require(msg.sender == _ownerAddress);
        _neuronInstance = Neuron(nrnAddress);
    }         

147  function mint(uint256 tokenId, uint256 quantity) external {
        require(tokenId < _itemCount);    

185  function setAllowedBurningAddresses(address newBurningAddress) public        
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L108
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L117
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L126
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L139
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L147
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L185