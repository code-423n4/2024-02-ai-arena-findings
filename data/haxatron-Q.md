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