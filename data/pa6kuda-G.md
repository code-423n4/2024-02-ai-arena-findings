# Remove unnecessary storage read from `VoltageManager.useVoltageBattery` when emmiting event `VoltageRemaining`

### Relevant GitHub Links

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L91-L99

## Summary

Function `VoltageManager.useVoltageBattery` replenishes voltage for a player, and emits an event that represents the remaining amount of voltage that player has. As this function sets remaining voltage of player to `100` there is no need to read `ownerVoltage[msg.sender]` when event emits.

## Recommended Mitigation Steps


Change the second argument in `VoltageRemaining` in `VoltageManager.useVoltageBattery` from `ownerVoltage[msg.sender]` to `100`.

```diff
  function useVoltageBattery() public {
      require(ownerVoltage[msg.sender] < 100);
      require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
      _gameItemsContractInstance.burn(msg.sender, 0, 1);
      ownerVoltage[msg.sender] = 100;
-     emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
+     emit VoltageRemaining(msg.sender, 100);
  }
```