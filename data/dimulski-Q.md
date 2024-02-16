[L-01] Wrong accounting of ``totalBattles``

```solidity
    /// @notice Number of total battles.
    uint256 public totalBattles = 0;
```

``totalBattles`` is increased each time [updateBattleRecord()](), the problem is that this will be called for each user, with the user specific result. This leads to a wrong accounting, as the ``totalBattles`` variable will represent double the count of the actual battles that took place. Consider modifiying the code as follows:
```diff
    function updateBattleRecord(
        uint256 tokenId, 
        uint256 mergingPortion,
        uint8 battleResult,
        uint256 eloFactor,
        bool initiatorBool
    ) 
        external 
    { 
        ...
        if (initiatorBool) {
            _voltageManagerInstance.spendVoltage(fighterOwner, VOLTAGE_COST);
+           totalBattles += 1;
        }
        /// INFO: This will be called for each user, not keeping correct track.
-        totalBattles += 1;
    }
```