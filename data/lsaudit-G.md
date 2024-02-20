# `AiArenaHelper.sol`

## `attributes` and `defaultAttributeDivisor` can be packed into single variable

The code-base does not provide any ability to extend `attributes` array. The list of attribute seems to be non-modifiable:

File: src/AiArenaHelper.sol
```
17:     string[] public attributes = ["head", "eyes", "mouth", "body", "hands", "feet"];
```

This suggests, that instead accessing every element of `attributes` - it will be more gas-efficient to pack this array into single `uint256` variable.

This would require multiple of refactoring steps, which we will describe here.
Firstly, we need to decide how `attributes` will be packed into array. Let's define, that each byte will represent different attribute, e.g:
```
1 - 000001 -> feet
2 - 000010  -> hands
4 - 000100 -> body
8 - 001000 -> mouth
16 - 010000 -> eyes
32 - 100000 -> head
```

This basically means, that `attributes` will be of type `uint256`: `uint256 public attributes`.
Updating the `attributes` type would require us to update other mappings, e.g., `attributeToDnaDivisor` will be changed from `mapping(string => uint8)` to `mapping(uint256 => uint8)`.

Now, the most interesting part - almost every function loops over `attributes.length`. We need to re-implement these loops. E.g., let's take a look at loop in `addAttributeDivisor()` function:

```
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeToDnaDivisor[attributes[i]] = attributeDivisors[i];
        }
```

we can easily rewrite that loop into a version when we'll be unpacking `attributes`.

```
        for (uint8 i = 0; i < 6; i++) {
            attributeToDnaDivisor[1 << i] = attributeDivisors[i];
        }
```


Similarly, loops in other functions needs to be rewritten.
That way, we will save a lot of gas - instead of using `string[] public attributes`, our attributes will be packed into single integer!


The same issue occurs with `defaultAttributeDivisor`. Since the length of this array is constant, we can pack it into a single `uint256`. Instead of using below array:

File: src/AiArenaHelper.sol
```
20:     uint8[] public defaultAttributeDivisor = [2, 3, 5, 7, 11, 13];
```

every value can be packed into a single `uint256`.

## Every occurrence of `attributes.length` can be pre-calculated

The code-base does not provide any ability to extend `attributes` array. The list of attribute seems to be non-modifiable:

File: src/AiArenaHelper.sol
```
17:     string[] public attributes = ["head", "eyes", "mouth", "body", "hands", "feet"];
```

This suggests, that the length of this array is constant - 6.
Instead of using `attributes.length` in the `AIArenaHelper.sol` - every occurrence of `attributes.length` can be changed to a constant value: 6.

E.g.:

File: src/AiArenaHelper.sol
```
147:         uint256 attributesLength = attributes.length;
148:         for (uint8 i = 0; i < attributesLength; i++) {
149:             attributeProbabilities[generation][attributes[i]] = new uint8[](0);
150:         }
```

can be changed to:

```
        for (uint8 i = 0; i < 6; i++) {
            attributeProbabilities[generation][attributes[i]] = new uint8[](0);
```

## `dnaToIndex()` can be optimized to return earlier

File: src/AiArenaHelper.sol
```
178:         for (uint8 i = 0; i < attrProbabilitiesLength; i++) {
179:             cumProb += attrProbabilities[i];
180:             if (cumProb >= rarityRank) {
181:                 attributeProbabilityIndex = i + 1;
182:                 break;
183:             }
184:         }
185:         return attributeProbabilityIndex;
```

As soon as we `break` from the loop, function returns with `attributeProbabilityIndex`. This implies, that we can immediately return inside the loop, instead of wasting gas on `break` operation and redundant `attributeProbabilityIndex = i + 1` assignment:

```
for (uint8 i = 0; i < attrProbabilitiesLength; i++) {
            cumProb += attrProbabilities[i];
            if (cumProb >= rarityRank) {
                return i + 1;
            }
```

## `attributes.length` should be cached earlier

The current implementation caches the `attributes.length` before the loop, so that it won't be calculated during every loop iteration. However, the length of `attributes` can be calculated earlier, since it's used in `require` statement.

File: src/AiArenaHelper.sol
```
70:         require(attributeDivisors.length == attributes.length);  // @audit: cache attributes.length here, so that cached variable will be used both in require and in a loop
71: 
72:         uint256 attributesLength = attributes.length;
73:         for (uint8 i = 0; i < attributesLength; i++) {
```

Our recommendation is to rewrite above code to:

```
        uint256 attributesLength = attributes.length;
        require(attributeDivisors.length == attributesLength);
        for (uint8 i = 0; i < attributesLength; i++) {
```

The same issue occurs in `createPhysicalAttributes()`

File: src/AiArenaHelper.sol
```
96:             uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributes.length); // @audit: cache attributes.length here, so that cached variable will be used both in require and in a loop
97: 
98:             uint256 attributesLength = attributes.length;
```

## `require` statement which uses less gas should be used first

File: src/AiArenaHelper.sol
```
132:         require(msg.sender == _ownerAddress);
133:         require(probabilities.length == 6, "Invalid number of attribute arrays");
```

Reading state variable `_ownerAddress` uses more gas than reading function's parameter `probabilities`, thus the order of require statements should be changed.


# `FighterFarm.sol`

## Do not calculate the length of arrays multiple of times

Calculating the length of arrays costs gas. It's better to calculate it once and cache the result.
Moreover, since we want to make sure that every parameters length is the same, it will be much better idea to calculate the length of the first parameter, cache it and compare it with the length of the others. That way we won't be calculating the length of other parameter twice.

In other words:

```
if (A.length == B.length &&
    B.length == C.length &&
    C.length == D.length &&
    D.length == E.length)
```

is the same as:

```
a = A.length
if (a == B.length &&
    a == C.length &&
    a == D.length &&
    a == E.length)
```

Below code:

File: src/FighterFarm.sol
```
243:         require(
244:             mintpassIdsToBurn.length == mintPassDnas.length && 
245:             mintPassDnas.length == fighterTypes.length && 
246:             fighterTypes.length == modelHashes.length &&
247:             modelHashes.length == modelTypes.length
248:         );
249:         for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
```

can be changed to:

```
        uint256 cachedMintpassIdsToBurn = mintpassIdsToBurn.length;
        require(
            cachedMintpassIdsToBurn == mintPassDnas.length && 
            cachedMintpassIdsToBurn == fighterTypes.length && 
            cachedMintpassIdsToBurn == modelHashes.length &&
            cachedMintpassIdsToBurn == modelTypes.length
        );
        for (uint16 i = 0; i < cachedMintpassIdsToBurn; i++) {
```

## Redundant variables in `_createFighterBase()`


There's no need to declare `element`, `weight`, `newDna`, since they are used only once, the code can be simplified to:

```
    function _createFighterBase(
        uint256 dna, 
        uint8 fighterType
    ) 
        private 
        view 
        returns (uint256, uint256, uint256) 
    {
        return (dna % numElements[generation[fighterType]], dna % 31 + 65, (fighterType == 0 ? dna : uint256(fighterType)));
    }
```

## `reRoll()` can be optimized

State variable used multiple of times should be cached.
Local variables used only once do not need to be declared at all.
Below code demonstrates above optimizations:

```
    function reRoll(uint8 tokenId, uint8 fighterType) public {
        require(msg.sender == ownerOf(tokenId));
        uint256 cachedNumRerolls = numRerolls[tokenId]; // @audit: cache numRerolls[tokenId]
        require(cachedNumRerolls < maxRerollsAllowed[fighterType]);  // @audit: use cached numRerolls
        uint256 cachedRerollCost = rerollCost; // @audit: cache rerollCost
        require(_neuronInstance.balanceOf(msg.sender) >= cachedRerollCost, "Not enough NRN for reroll"); // @audit: use cached rerollCost 

        _neuronInstance.approveSpender(msg.sender, cachedRerollCost);  // @audit: use cached rerollCost 
        if (_neuronInstance.transferFrom(msg.sender, treasuryAddress, cachedRerollCost)) { // @audit: use cached rerollCost, do not declare local var used only once
            numRerolls[tokenId] = cachedNumRerolls + 1
            (uint256 element, uint256 weight, uint256 newDna) = _createFighterBase(uint256(keccak256(abi.encode(msg.sender, tokenId, cachedNumRerolls + 1)));, fighterType); // @audit: cached numRerolls, line above numRellors were increased by one; redundant variable dna was removed
            fighters[tokenId].element = element;
            fighters[tokenId].weight = weight;
            fighters[tokenId].physicalAttributes = _aiArenaHelperInstance.createPhysicalAttributes(
                newDna,
                generation[fighterType],
                fighters[tokenId].iconsType,
                fighters[tokenId].dendroidBool
            );
            _tokenURIs[tokenId] = "";
        }
    }    
```

## State variables in `_createNewFighter()` should be cached

Function `_createNewFighter()` reads state variable twice. Reading state variable costs more gas than caching that variable into local variable and read the local variable.

File: src/FighterFarm.sol
```
499:         if (customAttributes[0] == 100) {  // @audit: cache customAttributes[0]
[...]
503:             element = customAttributes[0];  // @audit: cache customAttributes[0]
[...]
512:             generation[fighterType],  // @audit: cache generation[fighterType]
[...]
524:                 generation[fighterType],  // @audit: cache generation[fighterType]
[...]
530:         FighterOps.fighterCreatedEmitter(newId, weight, element, generation[fighterType]);  // @audit: cache generation[fighterType]
[...]
```



# `FighterOps.sol`

## Struct `FighterCreated` can be more tightly packed

When we take a look at function `_createFighterBase()` from `FighterFarm.sol`, we can see, that the initial max. `weight` is `30 + 65`:

File: src/FighterFarm.sol
```
471:         uint256 weight = dna % 31 + 65;
```

This suggests, that there's no need to declare `weight` as `uint256`, since it's very unlikely that this value ever reach `uint256`. The same reasoning goes with the `element`.
This implies, that below struct can be repacked from:

File: src/FighterOps.sol
```
36:     struct Fighter {
37:         uint256 weight;
38:         uint256 element;
39:         FighterPhysicalAttributes physicalAttributes;
40:         uint256 id;
41:         string modelHash;
42:         string modelType;
43:         uint8 generation;
44:         uint8 iconsType;
45:         bool dendroidBool;
46:     }
```

to:

```
    struct Fighter {
        uint128 weight;
        uint128 element;
        FighterPhysicalAttributes physicalAttributes;
        uint256 id;
        string modelHash;
        string modelType;
        uint8 generation;
        uint8 iconsType;
        bool dendroidBool;
    }
```

# `GameItems.sol`

## Consider packing `GameItemAttributes` struct more tightly

It's very unlikely that `itemPrice` will ever reach the max uint256 value. Consider changing its type to something smaller, e.g., `uint128`, so that it can be packed with `bool` fields, thus use one less storage slot:

```
    struct GameItemAttributes {
        string name;
        bool finiteSupply;
        bool transferable;
        uint128 itemPrice;
        uint256 itemsRemaining;
        uint256 dailyAllowance;
    }  
```

## Revert when `quantity` is 0 in `mint()`

Whenever `quantity` parameter is passed as 0 - the `price` will be calculated as 0 too, because: `price = allGameItemAttributes[tokenId].itemPrice * quantity`, thus `price = allGameItemAttributes[tokenId].itemPrice * 0`.
This implies, that we won't mint anything, but just use gas on useless operations (nothing will be minted). Consider adding additional `require` statement which will protect us from that scenario:

`require(quantity > 0, "Nothing to mint")`.

## Use post-increment to limit the number of state variable readings.

File: src/GameItems.sol
```
230:         if (!transferable) {
231:           emit Locked(_itemCount);
232:         }
233:         setTokenURI(_itemCount, tokenURI);
234:         _itemCount += 1;
235:     }
```

Variable `_itemCount` is used 3 times. Since it's a state variable we can limit a number of its readings by using post-incrementing. Above code can be rewritten to:

```
        if (!transferable) {
          emit Locked(_itemCount);
        }
        setTokenURI(_itemCount++, tokenURI);
```

## Remove redundant assignment and variable declaration in `getAllowanceRemaining()`

File: src/GameItems.sol
```
268:     function getAllowanceRemaining(address owner, uint256 tokenId) public view returns (uint256) {
269:         uint256 remaining = allowanceRemaining[owner][tokenId];
270:         if (dailyAllowanceReplenishTime[owner][tokenId] <= block.timestamp) {  
271:             remaining = allGameItemAttributes[tokenId].dailyAllowance;
272:         }
273:         return remaining;
274:     }
```

Above code can be rewritten to get rid of `remaining` declaration and redundant assignment: `remaining = allGameItemAttributes[tokenId].dailyAllowance`:

```
    function getAllowanceRemaining(address owner, uint256 tokenId) public view returns (uint256) {
        if (dailyAllowanceReplenishTime[owner][tokenId] <= block.timestamp) {  
            return allGameItemAttributes[tokenId].dailyAllowance;
        }
        return allowanceRemaining[owner][tokenId];
    }
```

# `MerginPool.sol`

## Do not loop over the whole `winnersLength` in `claimRewards()` and `getUnclaimedRewards`

According to the documentation - each round may have multiple of winners (chosen by `pickWinner()` function), however, the same address cannot be set as the winner multiple of times during the same round. This implies, that each rounds have multiple of unique winners.

File: src/MergingPool.sol
```
152:             for (uint32 j = 0; j < winnersLength; j++) {
153:                 if (msg.sender == winnerAddresses[currentRound][j]) {
154:                     _fighterFarmInstance.mintFromMergingPool(
155:                         msg.sender,
156:                         modelURIs[claimIndex],
157:                         modelTypes[claimIndex],
158:                         customAttributes[claimIndex]
159:                     );
160:                     claimIndex += 1;
161:                 }
```

This implies, that in the second loop at line 152, when we already find that `msg.sender == winnerAddresses[currentRound][j]`, we don't need to continue looping over `winnersLength` - since we have already found that `msg.sender == winnerAddresses[currentRound][j]`, thus we can break that loop.
Our recommendation is to add `break` after `claimIndex += 1;` at line 160. Since we've already found that `msg.sender` is the winner in the current loop, there's no need to continue looping over `winnersLength`. We can break for that loop and continue to look for a winner in another round (external loop at line 149).

The same issue occurs in `getUnclaimedRewards()`

File: src/MergingPool.sol
```
176:         for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
177:             winnersLength = winnerAddresses[currentRound].length;
178:             for (uint32 j = 0; j < winnersLength; j++) {
179:                 if (claimer == winnerAddresses[currentRound][j]) {
180:                     numRewards += 1;
181:                 }
182:             }
183:         }
```

When we found that `claimer == winnerAddresses[currentRound][j]` in loop at line 178, we can increase `numRewards` (`numRewards += 1`), break the loop and continue looping in loop at line 176:

```
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            winnersLength = winnerAddresses[currentRound].length;
            for (uint32 j = 0; j < winnersLength; j++) {
                if (claimer == winnerAddresses[currentRound][j]) {
                    numRewards += 1;
                    break;
                }
            }
        }
```

# `Neuron.sol`

## Use `_grantRole()` instead of `_setupRole()`.

Since, according to OZ's ERC-20 implementation, function `_setupRole()` is deprecated - `Neuron.sol` should use `_grantRole()` instead.
The `_setupRole()` implementation in OZ's ERC-20 implementation, directly calls `_grantRole()`:

```
    function _setupRole(bytes32 role, address account) internal virtual {
        _grantRole(role, account);
    }
```

This implies, that by calling `_setupRole()`, we waste gas on additional call. It's more gas efficient to call `_grantRole()` directly.
Below instances should use `_grantRole()` instead of `_setupRole()`.

File: src/Neuron.sol
```
093:     function addMinter(address newMinterAddress) external {
094:         require(msg.sender == _ownerAddress);
095:         _setupRole(MINTER_ROLE, newMinterAddress);
096:     }
[...]
101:     function addStaker(address newStakerAddress) external {
102:         require(msg.sender == _ownerAddress);
103:         _setupRole(STAKER_ROLE, newStakerAddress);
104:     }
[...]
109:     function addSpender(address newSpenderAddress) external {
110:         require(msg.sender == _ownerAddress);
111:         _setupRole(SPENDER_ROLE, newSpenderAddress);
112:     }
```

## `claim()` should check for 0-amount

Function `claim()` performs `transferFrom()`. However, it does not check if `amount` is non-zero. Whenever `amount` is 0, transferring `0` does basically nothing - since transferring 0-amount does not change the state of the blockchain, it's basically useless.
To save gas, our recommendation is to verify if `amount > 0`, before performing `tranferFrom()`.

Add additional `require` statement: `require(amount > 0, "Nothing to claim");`.

## No need to check for `allowance()` in `claim()`

File: src/Neuron.sol
```
138:     function claim(uint256 amount) external {
139:         require(
140:             allowance(treasuryAddress, msg.sender) >= amount, 
141:             "ERC20: claim amount exceeds allowance"
142:         );
143:         transferFrom(treasuryAddress, msg.sender, amount);
144:         emit TokensClaimed(msg.sender, amount);
145:     }
```

Function `transferFrom()` already checks the allowance. When we look at OZ's implementation of `transferFrom()`, it already calls `_spendAllowance()`. This implies, that there's no need to call additional `allowance()` in `claim()`. If claim amount exceeds allowance - function will revert on `transferFrom()`, thus lines 139-142 are redundant and can be removed.

## Cache `allowance()` result in `burnFrom()`

In function `burnFrom()`, function `allowance()` is called twice. It's much better idea to call it once and cache its result in local variable:

```
    function burnFrom(address account, uint256 amount) public virtual {
        uint256 cacheAllowance = allowance(account, msg.sender);
        require(
            cacheAllowance >= amount, 
            "ERC20: burn amount exceeds allowance"
        );
        uint256 decreasedAllowance = cacheAllowance - amount;
        _burn(account, amount);
        _approve(account, msg.sender, decreasedAllowance);
    }
```

## Do not declare redundant variables

In function `burnFrom()`, variable `decreasedAllowance` is used only once, thus there's no need to declare it at all:

```
 _approve(account, msg.sender, allowance(account, msg.sender) - amount);
```

# `StakeAtRisk.sol`

## `reclaimNRN()` could be optimized

`roundId` is a state variable, thus reading it costs more gas. Function `reclaimNRN()` reads this variable 3 times, while function `updateAtRiskRecords()` reads it twice.
Our recommendation is to cache `roundId` into local variable to avoid reading a state variable multiple of times:

```
    function reclaimNRN(uint256 nrnToReclaim, uint256 fighterId, address fighterOwner) external {
       [...]
       uint256 cachedRoundId = roundId;
        require(
            stakeAtRisk[cachedRoundId][fighterId] >= nrnToReclaim, 
[...]
            stakeAtRisk[cachedRoundId][fighterId] -= nrnToReclaim;
            totalStakeAtRisk[cachedRoundId] -= nrnToReclaim;
```

```
    function updateAtRiskRecords(
[...]
        uint256 cachedRoundId = roundId;
        stakeAtRisk[cachedRoundId][fighterId] += nrnToPlaceAtRisk;
        totalStakeAtRisk[cachedRoundId] += nrnToPlaceAtRisk; 
```

## Redundant `success` variables

There's no need to initialize `bool success` variable. Code like this:

```
bool success = someFunction();
if (success)
```

can be rewritten to `if(someFunction())`. That way, we can avoid declaration of additional variable.

File: src/StakeAtRisk.sol
```
80:         bool success = _sweepLostStake();
81:         if (success) {
```

can be changed to: `if (_sweepLostStake())`

File: src/StakeAtRisk.sol
```
100:         bool success = _neuronInstance.transfer(_rankedBattleAddress, nrnToReclaim);
101:         if (success) {
```

can be changed to: `if (_neuronInstance.transfer(_rankedBattleAddress, nrnToReclaim))`

# `RankedBattle.sol`

## `setNewRound()` could be optimized

File: src/RankedBattle.sol
```
233:     function setNewRound() external {
234:         require(isAdmin[msg.sender]);
235:         require(totalAccumulatedPoints[roundId] > 0);
236:         roundId += 1;
237:         _stakeAtRiskInstance.setNewRound(roundId);
238:         rankedNrnDistribution[roundId] = rankedNrnDistribution[roundId - 1];
239:     }
```

Since `roundId` is a state variable, accessing it multiple of times cost a lot of gas. Our recommendation is to cache this variable. Moreover, since at line 236 `roundId` is increased by one - we will use post-incrementing our cached variable (at line 235) - to avoid performing addition twice. If we didn't do that, we would be needed to perform additional addition operation at line 237. We can. however, avoid that as it's presented in the example below:

```
    function setNewRound() external {
        uint256 cachedRoundId = roundId;
        require(isAdmin[msg.sender]);
        require(totalAccumulatedPoints[cachedRoundId++] > 0);  
        roundId  = cachedRoundId;
        _stakeAtRiskInstance.setNewRound(cachedRoundId);
        rankedNrnDistribution[cachedRoundId] = rankedNrnDistribution[cachedRoundId - 1];
    }
```

##  `unstakeNRN()` can be optimized

Multiple of optimizations can be implemented to reduce the gas-intake of `unstakeNRN()`. In our report, we present all of them, including the final implementation.
In `unstakeNRN()`, `amountStaked[tokenId]` is a state variable which is read multiple of times. We can reduce number of readings by caching this variable.
The same goes with `roundId` - which can also be cached.
Moreover, we do not need to declare variable used only once (e.g., `success`). 
Since `if` condition guarantees that `amount > amountStaked[tokenId]`, `amountStaked[tokenId] -= amount` can be unchecked. The same with `globalStakedAmount -= amount`, since `globalStakedAmount >= amountStaked[tokenId]`
Our recommendation is to re-write `unstakeNRN()` as below:

```
   function unstakeNRN(uint256 amount, uint256 tokenId) external {  
        require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");
        uint256 cachedAmountStaked = amountStaked[tokenId];  // @audit: caching amountStaked[tokenId]
        uint256 cachedRoundId = roundId;  // @audit: caching roundId
        if (amount > cachedAmountStaked) {  // @audit: using cachedAmountStaked
            amount = cachedAmountStaked; // @audit: using cachedAmountStaked
        }
        unchecked {  // @audit, if condition guarantees this won't revert
        amountStaked[tokenId] -= amount;
        globalStakedAmount -= amount;
        }
        stakingFactor[tokenId] = _getStakingFactor(
            tokenId, 
            _stakeAtRiskInstance.getStakeAtRisk(tokenId)
        );
        _calculatedStakingFactor[tokenId][cachedRoundId] = true;  // @audit: using cached cachedRoundId
        hasUnstaked[tokenId][cachedRoundId] = true;  // @audit: using cached cachedRoundId
        if (_neuronInstance.transfer(msg.sender, amount)) {  // @ audit: get rid of redundant variable success
            if (cachedAmountStaked - amount == 0) {  // @audit: using cachedAmountStaked, since we have updated `amountStaked[tokenId] -= amount` (line 275) and cachedAmountStaked contains old value, we need to sub. amount.
                _fighterFarmInstance.updateFighterStaking(tokenId, false);
            }
            emit Unstaked(msg.sender, amount);
        }
    }
```

#  `VoltageManager.sol`

## Use `_replenishVoltage()` directly in `spendVoltage()` - to decrease a number of state variable readings

Using `_replenishVoltage()` code directly in `spendVoltage()`, would allow to cache state variables.
In the current implementation, we read state variable `ownerVoltage` 3 times (twice in `spendVoltage()` and once in `_replenishVoltage()`). Using code from `_replenishVoltage()` directly in `spendVoltage()` would allow us to get rid of one state variable reading, whenever the replenish needs to be done.

Whenever replenish needs to be done, we know that the voltage remaining after the replenish is always `100 - voltageSpent`. That way, we don't need to read `ownerVoltage[spender]` state variable again in the `VoltageRemaining` event.

Below code demonstrates the solution which saves us from additional state variable reading when replenish is needed.

File: src/VoltageManager.sol
```
   function spendVoltage(address spender, uint8 voltageSpent) public {
        require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
        if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
            ownerVoltage[spender] = 100 - voltageSpent;
            ownerVoltageReplenishTime[spender] = uint32(block.timestamp + 1 days);
            emit VoltageRemaining(spender, 100 - voltageSpent);  // @audit: saving gas here, using `100 - voltageSpent` instead of reading `ownerVoltage[spender]` again
        }
        else {
            ownerVoltage[spender] -= voltageSpent; 
            emit VoltageRemaining(spender, ownerVoltage[spender]);
        }
        
    }
```

## `useVoltageBattery()` is reading state variable, while its value is known

File: src/VoltageManager.sol
```
97:         ownerVoltage[msg.sender] = 100;
98:         emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
```

`ownerVoltage[msg.sender]` is assigned to 100, thus `ownerVoltage[msg.sender]` in `VoltageRemaining()` event will always return 100. We can get rid of reading state variable and hard-code 100 into that event:

```
        ownerVoltage[msg.sender] = 100;
        emit VoltageRemaining(msg.sender, 100);
```
