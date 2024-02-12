# [L01]: Users can accidentally waste a voltage battery even if the voltage is fully replenished after the replenish time is over

Observe the following code:

[VoltageManager.sol#L91-L99](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L91-L99)
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

To prevent users from wasting a voltage battery it does a check that `ownerVoltage[msg.sender] < 100` but if the battery is ready to replenish such that `ownerVoltageReplenishTime[spender] <= block.timestamp`. However, a user can still accidentally waste a voltage battery if they call `useVoltageBattery` when they could call `spendVoltage` which would immediately replenish their battery if `ownerVoltageReplenishTime[spender] <= block.timestamp`.

# [L02]: `deleteAttributeProbabilities` is redundant and can cause additional problems if accidentally called.

The function `deleteAttributeProbabilities` is redundant because an admin can already modify the probabilities via `addAttributeProbabilities` function.

[AiArenaHelper.sol#L141-L151](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L141C1-L151C6)
```solidity
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

Furthermore, it can cause additional problems if accidentally called. For instance if the admin calls `deleteAttributeProbabilities` on an active generation the `attrProbabilities` array will be set to 0. This will cause the output of `dnaToIndex` to be 0 which is invalid and can cause issues with the offchain mechanisms.

#[L03]: `totalAccumulatedPoints[roundId] > 0` in order to move to next round.

In order to move the round forward we require `totalAccumulatedPoints[roundId] > 0`
```solidity
    /// @notice Sets a new round, making claiming available for that round.
    /// @dev Only admins are authorized to move the round forward.
    function setNewRound() external {
        require(isAdmin[msg.sender]);
        require(totalAccumulatedPoints[roundId] > 0);
        roundId += 1;
        _stakeAtRiskInstance.setNewRound(roundId);
        rankedNrnDistribution[roundId] = rankedNrnDistribution[roundId - 1];
    }
``` 
In the extreme scenario where no one has accumulated any points and also has `stakeAtRisk`, admin cannot move round forward as `totalAccumulatedPoints[roundId] > 0`