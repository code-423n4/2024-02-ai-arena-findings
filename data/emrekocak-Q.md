# L-1) `totalBattles` calculation is wrong
## Impact
Since `RankedBattle::updateBattleRecord` runs twice for each battle, `totalBattles` increases by two for every battle. 
## Recommended Mitigation Steps
If in every battle just one player pays voltage, move the `totalBattles += 1;` line inside this if statement. 
```diff
diff --git a/src/RankedBattle.sol#L345-L348
        if (initiatorBool) {
            _voltageManagerInstance.spendVoltage(fighterOwner, VOLTAGE_COST);
+           totalBattles += 1;
        }
-       totalBattles += 1;
```