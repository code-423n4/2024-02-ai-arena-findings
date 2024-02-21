## Summary

In the VoltageManager.sol contract, useVoltageBattery() function checks if user has 100 voltage, and if not, consumes a battery and restores voltage to 100:

    function useVoltageBattery() public {
        require(ownerVoltage[msg.sender] < 100);
        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
        _gameItemsContractInstance.burn(msg.sender, 0, 1);
        ownerVoltage[msg.sender] = 100;
        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
    }


Presumably, a user would do this if they want to spend their voltage and they have none. The 100 check makes sure that a user doesn't waste their battery if they're already at 100 voltage.

The only way to spend voltage is through the spendVoltage function:

    function spendVoltage(address spender, uint8 voltageSpent) public {
        require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
        if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
            _replenishVoltage(spender);
        }
        ownerVoltage[spender] -= voltageSpent;
        emit VoltageRemaining(spender, ownerVoltage[spender]);
    }

This function checks if replenishing is possible and, if so, does it before spending voltage.

    function _replenishVoltage(address owner) private {
        ownerVoltage[owner] = 100;
        ownerVoltageReplenishTime[owner] = uint32(block.timestamp + 1 days);
    }  

This timed voltage replenishment is functionally the same a using a battery.

As it is now, the useVoltageBattery function smartly check if user is already at maximum voltage, but it does not check if replenishment is possible.
If replenishment is possible and a user uses a battery, their voltage will get capped and replenish time reset anyway as soon as they try to spend their voltage - making the battery use a waste.

## Impact

User negligence may lead to loss of a valuable asset, which can be prevented by the protocol.

## Recommended Mitigation Steps

Add a new check to the useVoltageBattery function:

    require(ownerVoltageReplenishTime[spender] > block.timestamp);