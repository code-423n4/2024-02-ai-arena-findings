## [NC-01] No point in allowing user to directly call `VoltageManager::spendVoltage` function.

The `spendVoltage` function is as follows ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L105 ):

```solidity
    /// @notice Spends voltage.
    /// @dev This function can be called by the user or an allowed voltage spender.
    /// @param spender The address of the spender.
    /// @param voltageSpent The amount of voltage to spend.
    function spendVoltage(address spender, uint8 voltageSpent) public {
        require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
        if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
            _replenishVoltage(spender);
        }
        ownerVoltage[spender] -= voltageSpent;
        emit VoltageRemaining(spender, ownerVoltage[spender]);
    }
```

If user call this function directly then they will lose their voltage without any benefit. So, it's better to not let the user call this function directly.

## [NC-02] Calling `VoltageManager::useVoltageBattery` if `VOLTAGE_COST < voltage < 100` would waste the excess voltage.

The `useVoltageBattery` function is as follows ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L93 ):

```solidity
    /// @notice Uses a voltage battery to replenish voltage for a player in AI Arena.
    /// @dev This function can be called by any user to use the voltage battery.
    function useVoltageBattery() public {
        require(ownerVoltage[msg.sender] < 100);
        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
        _gameItemsContractInstance.burn(msg.sender, 0, 1);
        ownerVoltage[msg.sender] = 100;
        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
    }
```

If the user has voltage less than 100 but greater than `VOLTAGE_COST` then they will lose the excess voltage. It's better to let the user use the excess voltage or to not let them call this function unless they have voltage less than `VOLTAGE_COST` i.e `10` ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L53 ).

For example, if the user has 50 voltage and they call `useVoltageBattery` then their voltage will be set to 100 and they will lose 50 voltage, which they could have used in the game. Compare this with the case where the user has 5 voltage and they call `useVoltageBattery` then their voltage will be set to 100 and they will lose 5 voltage, which is fine because they can't use 5 voltage in the game.

## [NC-03] Revert if `_neuronInstance::transfer` or `_neuronInstance::transferFrom` fails.

The `_neuronInstance::transfer` or `_neuronInstance::transferFrom` function always returns `true` but still it's better to only consider the `success` case but also the `failure` case.

<details>

<summary>
There are three instances which only consider the `success` case but not the `failure` case:

</summary>

###

#

`RankedBattle::stakeNRN` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L244 ):

```solidity
function stakeNRN(uint256 amount, uint256 tokenId) external {

    ...

@-> bool success = _neuronInstance.transferFrom(msg.sender, address(this), amount);
    if (success) {

        ...

    }
}
```

#

`RankedBattle::unstakeNRN` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L270 ):

```solidity
function unstakeNRN(uint256 amount, uint256 tokenId) external {

    ...

@-> bool success = _neuronInstance.transfer(msg.sender, amount);
    if (success) {

        ...

    }
}
```

#

`RankedBattle::_addResultPoints` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L416 ):

```solidity
function _addResultPoints(
    uint8 battleResult,
    uint256 tokenId,
    uint256 eloFactor,
    uint256 mergingPortion,
    address fighterOwner
)
    private
{

    ...

    if (battleResult == 0) {
        /// If the user won the match
        ...

    } else if (battleResult == 2) {
        /// If the user lost the match

        ...

        if (accumulatedPointsPerFighter[tokenId][roundId] > 0) {
            /// If the fighter has a positive point balance for this round, deduct points

            ...

        } else {
            /// If the fighter does not have any points for this round, NRNs become at risk of being lost
@->         bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
            if (success) {

                ...

            }
        }
    }
}
```

It is better to revert if `_neuronInstance::transfer` or `_neuronInstance::transferFrom` fails.

</details>

## [NC-04] It's better to check for `zero` value while updating state variables.

<details>

<summary>
The instances having this issue:

</summary>

###

#

The `VoltageManager::transferOwnership` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L64 ):

```solidity
/// @notice Transfers ownership from one address to another.
/// @dev Only the owner address is authorized to call this function.
/// @param newOwnerAddress The address of the new owner
function transferOwnership(address newOwnerAddress) external {
    require(msg.sender == _ownerAddress);
    _ownerAddress = newOwnerAddress;
}
```

It should be checked that `newOwnerAddress` is not `address(0)`.

#

The `RankedBattle::setBpsLostPerLoss` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L226 ):

```solidity
/// @notice Sets the basis points lost per ranked match lost while in a point deficit.
/// @dev Only admins are authorized to call this function.
/// @param bpsLostPerLoss_ The basis points lost per loss.
function setBpsLostPerLoss(uint256 bpsLostPerLoss_) external {
    require(isAdmin[msg.sender]);
    bpsLostPerLoss = bpsLostPerLoss_;
}
```

It should be checked that `bpsLostPerLoss_` is not `0`.

#

The `RankedBattle::setRankedNrnDistribution` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L218 ):

```solidity
/// @notice Sets the ranked nrn distribution amount for the current round.
/// @dev Only admins are authorized to change the ranked NRN distribution.
/// @dev newDistribution is NOT denominated in wei. As such, it is multiplied by 10**18.
/// @param newDistribution The new distribution amount.
function setRankedNrnDistribution(uint256 newDistribution) external {
    require(isAdmin[msg.sender]);
    rankedNrnDistribution[roundId] = newDistribution * 10**18;
}
```

It should be checked that `newDistribution` is not `0`.

#

The `RankedBattle::instantiateMergingPoolContract` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L209 ):

```solidity
/// @notice Instantiates the merging pool contract.
/// @dev Only the owner address is authorized to call this function.
/// @param mergingPoolAddress The address of the MergingPool contract.
function instantiateMergingPoolContract(address mergingPoolAddress) external {
    require(msg.sender == _ownerAddress);
    _mergingPoolInstance = MergingPool(mergingPoolAddress);
}
```

It should be checked that `mergingPoolAddress` is not `address(0)`.

#

The `RankedBattle::instantiateNeuronContract` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L201 ):

```solidity
/// @notice Instantiates the neuron contract.
/// @dev Only the owner address is authorized to call this function.
/// @param nrnAddress The address of the Neuron contract.
function instantiateNeuronContract(address nrnAddress) external {
    require(msg.sender == _ownerAddress);
    _neuronInstance = Neuron(nrnAddress);
}
```

It should be checked that `nrnAddress` is not `address(0)`.

#

The `RankedBattle::setStakeAtRiskAddress` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L192 ):

```solidity
/// @notice Sets the Stake at Risk contract address and instantiates the contract.
/// @dev Only the owner address is authorized to call this function.
/// @param stakeAtRiskAddress The address of the Stake At Risk contract.
function setStakeAtRiskAddress(address stakeAtRiskAddress) external {
    require(msg.sender == _ownerAddress);
    _stakeAtRiskAddress = stakeAtRiskAddress;
    _stakeAtRiskInstance = StakeAtRisk(_stakeAtRiskAddress);
}
```

It should be checked that `stakeAtRiskAddress` is not `address(0)`.

#

The `RankedBattle::setGameServerAddress` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L184 ):

```solidity
/// @notice Sets the game server address.
/// @dev Only the owner address is authorized to call this function.
/// @param gameServerAddress The game server address.
function setGameServerAddress(address gameServerAddress) external {
    require(msg.sender == _ownerAddress);
    _gameServerAddress = gameServerAddress;
}
```

It should be checked that `gameServerAddress` is not `address(0)`.

#

The `RankedBattle::transferOwnership` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L167 ):

```solidity
/// @notice Transfers ownership from one address to another.
/// @dev Only the owner address is authorized to call this function.
/// @param newOwnerAddress The address of the new owner
function transferOwnership(address newOwnerAddress) external {
    require(msg.sender == _ownerAddress);
    _ownerAddress = newOwnerAddress;
}
```

It should be checked that `newOwnerAddress` is not `address(0)`.

#

The `MergingPool::transferOwnership` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L89 ):

```solidity
/// @notice Transfers ownership from one address to another.
/// @dev Only the owner address is authorized to call this function.
/// @param newOwnerAddress The address of the new owner
function transferOwnership(address newOwnerAddress) external {
    require(msg.sender == _ownerAddress);
    _ownerAddress = newOwnerAddress;
}
```

It should be checked that `newOwnerAddress` is not `address(0)`.

</details>
