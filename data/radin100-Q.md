## [L-1] User is open to waste batteries trough 'VoltageManager.sol'
### CodeSnipped
https://github.com/code-423n4/2024-02-ai-arena/blob/70b73ce5acaf10bc331cf388d35e4edff88d4541/src/VoltageManager.sol#L93-L121
### PoC
Scenario:
1. User thinks he has no daily refill so he decides to buy and use battery
2. So user has maximum voltage again
3. User decides do practise in battle so he spend some voltage(first spend after more than a day after the last spend) and 'VoltageManager.sol::spendVoltage' is activated
4. 'VoltageManager.sol::spendVoltage' executes 'VoltageManager.sol::_replenishVoltage' which gets user's voltage at maximum amount for second time, which makes the voltage from the battery(used at the beginning) wasted

Proof of code:
Next lines of code(2 test functions) are executed at VoltageManager.t.sol:

     function testWastedBattery() public {
        address player = makeAddr("PLAYER");
        vm.prank(player);
        _voltageManagerContract.spendVoltage(player, 10); // after that player must have 90 voltage
        assertEq(_voltageManagerContract.ownerVoltage(player), 90);

        skip(1 days);

        _mintSingleBatteryToPlayer(player); // mit battery
        vm.startPrank(player);
        _voltageManagerContract.useVoltageBattery(); // this battery use is unnecessary but it is avilable voltage are still under 100
        _voltageManagerContract.spendVoltage(player, 10); // after that player must have 90 voltage again because there is after 1 day
        vm.stopPrank();

        assertEq(_voltageManagerContract.ownerVoltage(player), 90);
        assertEq(_gameItemsContract.balanceOf(player, 0), 0); // no batteries left
    }

    function testNoWastedBattery() public {
        address player = makeAddr("PLAYER");
        vm.prank(player);
        _voltageManagerContract.spendVoltage(player, 10); // after that player must have 90 voltage
        assertEq(_voltageManagerContract.ownerVoltage(player), 90);

        skip(1 days);

        _mintSingleBatteryToPlayer(player); // mit battery
        vm.startPrank(player);
        _voltageManagerContract.spendVoltage(player, 10); // after that player must have 90 voltage again because there is after 1 day
        vm.stopPrank();

        assertEq(_voltageManagerContract.ownerVoltage(player), 90);
        assertEq(_gameItemsContract.balanceOf(player, 0), 1); // single battery left
    }
### Recommendation
Add check at the beginning of 'VoltageManager.sol::useVoltageBattery' that makes sure no batteries is going to be wasted after next voltage spend:

    function useVoltageBattery() public {
       + if (ownerVoltageReplenishTime[msg.sender] <= block.timestamp) {
       +     return _replenishVoltage(msg.sender);
       + }
        require(ownerVoltage[msg.sender] < 100);
        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
        _gameItemsContractInstance.burn(msg.sender, 0, 1);
        ownerVoltage[msg.sender] = 100;
        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
    }