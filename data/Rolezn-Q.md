### QA Issues
| # |Issue|
|-|-|
| [QA&#x2011;1](#QA&#x2011;1) | Max rerolls can be increased until max `uint8`
| [QA&#x2011;2](#QA&#x2011;2) | Fighter generation is limited to max `uint8`
| [QA&#x2011;3](#QA&#x2011;3) | It is not possible to remove staker after calling `FighterFarm.addStaker()`
| [QA&#x2011;4](#QA&#x2011;4) | Missing length check for `FighterFarm.iconsTypes`
| [QA&#x2011;5](#QA&#x2011;5) | `FighterFarm.mintFromMergingPool()` always creates fighterType 0
| [QA&#x2011;6](#QA&#x2011;6) | Project uses an old version of `openzeppelin-contracts`
| [QA&#x2011;7](#QA&#x2011;7) | In `FighterFarm.reRoll()` `approveSpender` should be called with 0 `amount` if `_neuronInstance.transferFrom()` fails
| [QA&#x2011;8](#QA&#x2011;8) | `generator` has different `uint` sizes on different functions
| [QA&#x2011;9](#QA&#x2011;9) | It is possible to `addAttributeProbabilities()` exceeding `uint8` `generation` values but unable to `deleteAttributeProbabilities()` delete anything exceeding `uint8` `generation`
| [QA&#x2011;10](#QA&#x2011;10) | It is not possible to remove addresses from `allowedBurningAddresses` once added
| [QA&#x2011;11](#QA&#x2011;11) | `GameItems.mint()` should be called `buyGameItem()` rather than `mint()`
| [QA&#x2011;12](#QA&#x2011;12) | `GameItems.mint()` should emit an event when `tranferFrom` has failed and purchase was unsuccessful.
| [QA&#x2011;13](#QA&#x2011;13) | An `event` should be emitted when winners have been picked in `MergingPool.pickWinners()`
| [QA&#x2011;14](#QA&#x2011;14) | `RankedBattle.setStakeAtRiskAddress()` should be renamed `RankedBattle.instantiateStakeAtRiskContract()`
| [QA&#x2011;15](#QA&#x2011;15) | `VoltageManager.useVoltageBattery()` can be set to require only the `VOLTAGE_COST` to allow players to be more efficient with their battery usage
| [QA&#x2011;16](#QA&#x2011;16) | `VoltageManager.spendVoltage()` can be called by the player.
| [QA&#x2011;17](#QA&#x2011;17) | Use a `constant` for `NRN` decimals
| [QA&#x2011;18](#QA&#x2011;18) | Staked `NRN` on a fighter can be lost when NFT is transferred to another address
| [QA&#x2011;19](#QA&#x2011;19) | `MergingPool.pickWinner()` is susceptible to manual mistake calls

### <a href="#qa-summary">[QA&#x2011;1]</a><a name="QA&#x2011;1"> Max rerolls can be increased until max `uint8`

According to the initial value, the `maxRerollsAllowed` is set to 3 per `FighterType`

```solidity
/// @notice The maximum amount of rerolls for each fighter.
    uint8[2] public maxRerollsAllowed = [3, 3];
```

However, in `FighterFarm.sol` contract each time `incrementGeneration` function is called, it will increase the `maxRerollsAllowed` by 1. It is unclear if it is intended as the indication MAX usually means to limit a specific amount of rerolls, however by calling this function it is possible to increase the max until 255 value

```solidity

132: maxRerollsAllowed[fighterType] += 1;

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L132

Instead, each generation increment should reset back to its default max [3,3] value





### <a href="#qa-summary">[QA&#x2011;2]</a><a name="QA&#x2011;2"> Fighter generation is limited to max `uint8`

In `FighterFarm.sol` contract the `incrementGeneration` function limits the return to uint8.

```solidity

42: uint8[2] public generation = [0, 0];

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L42

When `incrementGeneration` is called, it increases the `generation` by 1.

```solidity
131: generation[fighterType] += 1;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L131

When the game will be deployed I believe that generation should easily exceed 255 overtime


### <a href="#qa-summary">[QA&#x2011;3]</a><a name="QA&#x2011;3"> It is not possible to remove staker after calling `FighterFarm.addStaker()`

It is not possible to remove staker after calling `FighterFarm.addStaker()`

`FighterFarm.addStaker()` allows adding stakers to allowing staking fighters on behalf of users, however it is not possible to remove them after adding them.

```solidity

function addStaker(address newStaker) external {
        require(msg.sender == _ownerAddress);
        hasStakerRole[newStaker] = true;
    }

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L139-L142

It is recommended to add a `removeStaker` option.
```solidity

function removeStaker(address staker) external {
        require(msg.sender == _ownerAddress);
        hasStakerRole[staker] = false;
    }

```

### <a href="#qa-summary">[QA&#x2011;4]</a><a name="QA&#x2011;4"> Missing length check for `FighterFarm.iconsTypes`

`FighterFarm.redeemMintPass()` checks for the input arrays length checks.

```solidity

require(
            mintpassIdsToBurn.length == mintPassDnas.length && 
            mintPassDnas.length == fighterTypes.length && 
            fighterTypes.length == modelHashes.length &&
            modelHashes.length == modelTypes.length
        );

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L243-L248

However, it is missing a length check for `iconsTypes`. This can result in an inaccessible array call when calling `FighterFarm._createNewFighter()` for an non-existing `iconsTypes` index.

```solidity

for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
            require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));
            _mintpassInstance.burn(mintpassIdsToBurn[i]);
            _createNewFighter(
                msg.sender, 
                uint256(keccak256(abi.encode(mintPassDnas[i]))), 
                modelHashes[i], 
                modelTypes[i],
                fighterTypes[i],
                iconsTypes[i],
                [uint256(100), uint256(100)]
            );
        }

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L249-L261

It is recommended to add a length check for `iconsTypes` as well, i.e.:

```solidity

require(
            mintpassIdsToBurn.length == mintPassDnas.length && 
            mintPassDnas.length == fighterTypes.length && 
            fighterTypes.length == modelHashes.length &&
            modelHashes.length == modelTypes.length &&
	 		modelTypes.length == iconsTypes.length
        );


```


### <a href="#qa-summary">[QA&#x2011;5]</a><a name="QA&#x2011;5"> `FighterFarm.mintFromMergingPool()` always creates fighterType 0

`FighterFarm.mintFromMergingPool()` always creates fighterType 0

It is unclear whether it is intended behavior as when calling `FighterFarm.mintFromMergingPool()`, it will always call `_createNewFighter()` with fighterType 0.

Since when looking at `claimFighters()` and `redeemMintPass()` it is possible to see that it is possible to create figherTypes of 0 and 1 unlike in `mintFromMergingPool()`

```solidity
        _createNewFighter(
            to, 
            uint256(keccak256(abi.encode(msg.sender, fighters.length))), 
            modelHash, 
            modelType,
            0,
            0,
            customAttributes
        );
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L322-L330



### <a href="#qa-summary">[QA&#x2011;6]</a><a name="QA&#x2011;6"> Project uses an old version of `openzeppelin-contracts`

According to https://github.com/code-423n4/2024-02-ai-arena/tree/main/lib

The project is using an `openzeppelin-contracts` version from 2 years of commit: `ecd2ca2cd7cac116f7a37d0e474bbb3d7d5e1c4d`

It is highly recommended to use the latest `openzeppelin-contracts` version as many of the functionalities may have changed.



### <a href="#qa-summary">[QA&#x2011;7]</a><a name="QA&#x2011;7"> In `FighterFarm.reRoll()` `approveSpender` should be called with 0 `amount` if `_neuronInstance.transferFrom()` fails

In `FighterFarm.reRoll()` the spender has to approve the amount of `rerollCost`.

```solidity

_neuronInstance.approveSpender(msg.sender, rerollCost);

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L375

Then a `transferFrom()` is performed to the `treasuryAddress` and is checked for `success`. However, if the transfer failed, the approve amount is still set to the `rerollCost`. 

```solidity

bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L376


It's best to call approve to `amount` 0, should the transfer fail.

```solidity

...
else {
	_neuronInstance.approveSpender(msg.sender, 0);
}

```



### <a href="#qa-summary">[QA&#x2011;8]</a><a name="QA&#x2011;8"> `generator` has different `uint` sizes on different functions

Either decide to use `uint256` or `uint8` for `generation`

```solidity

    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
    }

     /// @notice Delete attribute probabilities for a given generation. 
     /// @dev Only the owner can call this function.
     /// @param generation The generation number.
    function deleteAttributeProbabilities(uint8 generation) public {
        require(msg.sender == _ownerAddress);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = new uint8[](0);
        }
    }

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131-L151



### <a href="#qa-summary">[QA&#x2011;9]</a><a name="QA&#x2011;9"> It is possible to `addAttributeProbabilities()` exceeding `uint8` `generation` values but unable to `deleteAttributeProbabilities()` delete anything exceeding `uint8` `generation`

Looking at `AiArenaHelper.addAttributeProbabilities()`, we can see that it is possible to add `attributeProbabilities` to `generation` values of up to `uint256`

```solidity

function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
    }

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131-L139

However, when looking at `AiArenaHelper.deleteAttributeProbabilities()`, we can see that it is only possible to remove `attributeProbabilities` up to `uint8`

```solidity

    function deleteAttributeProbabilities(uint8 generation) public {
        require(msg.sender == _ownerAddress);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = new uint8[](0);
        }
    }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L144-L151


### <a href="#qa-summary">[QA&#x2011;10]</a><a name="QA&#x2011;10"> It is not possible to remove addresses from `allowedBurningAddresses` once added

Should an address that is no longer be authorized (i.e. due to compromise), it is not possible to remove an address from `allowedBurningAddresses`.

```solidity

    function setAllowedBurningAddresses(address newBurningAddress) public {
        require(isAdmin[msg.sender]);
        allowedBurningAddresses[newBurningAddress] = true;
    }


```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L185-L188



### <a href="#qa-summary">[QA&#x2011;11]</a><a name="QA&#x2011;11"> `GameItems.mint()` should be called `buyGameItem()` rather than `mint()`

Looking at the `mint()` functionality, we can see that it behaves as in similar to buying a game item rather than minting. It would be more relatable if the function would be called as a `buy` function rather than a `mint` function.

```solidity

    function mint(uint256 tokenId, uint256 quantity) external {
        require(tokenId < _itemCount);
        uint256 price = allGameItemAttributes[tokenId].itemPrice * quantity;
        require(_neuronInstance.balanceOf(msg.sender) >= price, "Not enough NRN for purchase");
        require(
            allGameItemAttributes[tokenId].finiteSupply == false || 
            (
                allGameItemAttributes[tokenId].finiteSupply == true && 
                quantity <= allGameItemAttributes[tokenId].itemsRemaining
            )
        );
        require(
            dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp || 
            quantity <= allowanceRemaining[msg.sender][tokenId]
        );

        _neuronInstance.approveSpender(msg.sender, price);
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
            emit BoughtItem(msg.sender, tokenId, quantity);
        }
    }


```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L147-L176



### <a href="#qa-summary">[QA&#x2011;12]</a><a name="QA&#x2011;12"> `GameItems.mint()` should emit an event when `tranferFrom` has failed and purchase was unsuccessful.

Currently the `mint()` function, will emit that an item has been bought if the transaction was a success, but does not emit when it fails. We should add a check where a transaction fails it should emit a failure.

```solidity

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
            emit BoughtItem(msg.sender, tokenId, quantity);
        }

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L164-L175



### <a href="#qa-summary">[QA&#x2011;13]</a><a name="QA&#x2011;13"> An `event` should be emitted when winners have been picked in `MergingPool.pickWinners()`

Currently, no event is emitted when winners have been picked or that `isSelectionComplete` is now true.
An event emitting the winners, isSelectionComplete and roundId should be emitted.

```solidity

    function pickWinner(uint256[] calldata winners) external {
        require(isAdmin[msg.sender]);
        require(winners.length == winnersPerPeriod, "Incorrect number of winners");
        require(!isSelectionComplete[roundId], "Winners are already selected");
        uint256 winnersLength = winners.length;
        address[] memory currentWinnerAddresses = new address[](winnersLength);
        for (uint256 i = 0; i < winnersLength; i++) {
            currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);
            totalPoints -= fighterPoints[winners[i]];
            fighterPoints[winners[i]] = 0;
        }
        winnerAddresses[roundId] = currentWinnerAddresses;
        isSelectionComplete[roundId] = true;
        roundId += 1;
    }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L118-L132



### <a href="#qa-summary">[QA&#x2011;14]</a><a name="QA&#x2011;14"> `RankedBattle.setStakeAtRiskAddress()` should be renamed `RankedBattle.instantiateStakeAtRiskContract()`

As similar to `instantiateNeuronContract` and `instantiateMergingPoolContract`, the `RankedBattle.setStakeAtRiskAddress()` behaves the same as the other functions and should be called similarly as well.

```solidity
    /// @notice Sets the Stake at Risk contract address and instantiates the contract.
    /// @dev Only the owner address is authorized to call this function.
    /// @param stakeAtRiskAddress The address of the Stake At Risk contract.
    function setStakeAtRiskAddress(address stakeAtRiskAddress) external {
        require(msg.sender == _ownerAddress);
        _stakeAtRiskAddress = stakeAtRiskAddress;
        _stakeAtRiskInstance = StakeAtRisk(_stakeAtRiskAddress);
    }

    /// @notice Instantiates the neuron contract.
    /// @dev Only the owner address is authorized to call this function.
    /// @param nrnAddress The address of the Neuron contract.
    function instantiateNeuronContract(address nrnAddress) external {
        require(msg.sender == _ownerAddress);
        _neuronInstance = Neuron(nrnAddress);
    }

    /// @notice Instantiates the merging pool contract.
    /// @dev Only the owner address is authorized to call this function.
    /// @param mergingPoolAddress The address of the MergingPool contract.
    function instantiateMergingPoolContract(address mergingPoolAddress) external {
        require(msg.sender == _ownerAddress);
        _mergingPoolInstance = MergingPool(mergingPoolAddress);
    }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L189-L212

```diff

-    function setStakeAtRiskAddress(address stakeAtRiskAddress) external {
+    function instantiateStakeAtRiskContract(address stakeAtRiskAddress) external {
         require(msg.sender == _ownerAddress);
-        _stakeAtRiskAddress = stakeAtRiskAddress;
-        _stakeAtRiskInstance = StakeAtRisk(_stakeAtRiskAddress);
+        _stakeAtRiskInstance = StakeAtRisk(stakeAtRiskAddress);
    }

```



### <a href="#qa-summary">[QA&#x2011;15]</a><a name="QA&#x2011;15"> `VoltageManager.useVoltageBattery()` can be set to require only the `VOLTAGE_COST` to allow players to be more efficient with their battery usage

Since initiating a ranked battle costs `VOLTAGE_COST = 10`, then we can require `ownerVoltage[msg.sender]` to be less than `10` in order to call `VoltageManager.useVoltageBattery()`. 
This will let players maximize the efficiency of their current voltage with the battery usage and to not waste unspent voltage.

```diff

    function useVoltageBattery() public {
-       require(ownerVoltage[msg.sender] < 100);
+       require(ownerVoltage[msg.sender] < 10);
        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
        _gameItemsContractInstance.burn(msg.sender, 0, 1);
        ownerVoltage[msg.sender] = 100;
        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
    }

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L93-L99



### <a href="#qa-summary">[QA&#x2011;16]</a><a name="QA&#x2011;16"> `VoltageManager.spendVoltage()` can be called by the player.

`VoltageManager.spendVoltage()` is allowed to be called by the players, there's no reason for `VoltageManager.spendVoltage()` to be called by players at any point.

```solidity
    function spendVoltage(address spender, uint8 voltageSpent) public {
        require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
        if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
            _replenishVoltage(spender);
        }
        ownerVoltage[spender] -= voltageSpent;
        emit VoltageRemaining(spender, ownerVoltage[spender]);
    }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L105-L112

The only usage of  `VoltageManager.spendVoltage()` is when a ranked battle is initiated by the `RankedBattle.updateBattleRecord()` call.

```solidity
	_voltageManagerInstance.spendVoltage(fighterOwner, VOLTAGE_COST);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L346



### <a href="#qa-summary">[QA&#x2011;17]</a><a name="QA&#x2011;17"> Use a `constant` for `NRN` decimals

It would be much more clearer and for readability purposes to set a `constant` for `NRN` decimals.

```diff

+   /// @notice NRN decimals
+   uint256 public constant NRN_DECIMALS = 10**18;

    /// @notice Initial supply of NRN tokens to be minted to the treasury.
-   uint256 public constant INITIAL_TREASURY_MINT = 10**18 * 10**8 * 2;
+   uint256 public constant INITIAL_TREASURY_MINT = NRN_DECIMALS * 10**8 * 2;

    /// @notice Initial supply of NRN tokens to be minted and distributed to contributors.
-   uint256 public constant INITIAL_CONTRIBUTOR_MINT = 10**18 * 10**8 * 5;
+   uint256 public constant INITIAL_CONTRIBUTOR_MINT =  NRN_DECIMALS * 10**8 * 5;

    /// @notice Maximum supply of NRN tokens.
-   uint256 public constant MAX_SUPPLY = 10**18 * 10**9;
+   uint256 public constant MAX_SUPPLY =  NRN_DECIMALS * 10**9;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L36-L43



### <a href="#qa-summary">[QA&#x2011;18]</a><a name="QA&#x2011;18"> Staked `NRN` on a fighter can be lost when NFT is transferred to another address

`amountStaked` is mapped according to the `tokenId` rather than the owner address

```solidity

    /// @notice Mapping of token id to staked amount.
    mapping(uint256 => uint256) public amountStaked;

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L127-L128

When transferring the NFT fighter to a different address via `FighterFarm.transferFrom()` or `FighterFarm.safeTransferFrom()`, the staked will not transfer to the previous owner.
There should be a warning regarding this when transferring NFT fighters.



### <a href="#qa-summary">[QA&#x2011;19]</a><a name="QA&#x2011;19"> `MergingPool.pickWinner()` is susceptible to manual mistake calls

`MergingPool.pickWinner()` is a used by an admin to manually call the function in order to pick winners for the current round.
When the winners are picked, each of the winner's points are deducted to 0 value and then the `roundId` is increased by 1.

```solidity

roundId += 1;

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L131

Since this is function is called manually by an admin, there is a possibly where an admin may mistakenly call it twice and as a result there will be no way to revert the selection of winners since the `roundId` is increased by 1 and there is no way to decrease it by 1 in the `MergingPool` contract.

```solidity

winnerAddresses[roundId] = currentWinnerAddresses;
        isSelectionComplete[roundId] = true;
        roundId += 1;

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L129-L131