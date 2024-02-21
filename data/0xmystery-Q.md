## [L-01] Incomplete array length check
`FighterFarm.redeemMintPass()`'s design requires equal lengths for arrays such as `mintpassIdsToBurn`, `mintPassDnas`, `fighterTypes`, `modelHashes`, and `modelTypes` to ensure each element corresponds across these arrays during the fighter creation process. However, the initial implementation overlooked the `iconsTypes` array, which also plays a significant role in this process. Incorporating a length check for the `iconsTypes` array alongside the existing ones is essential to guarantee that each fighter's creation is based on consistent and complete data, thereby enhancing the contract's reliability and preventing transaction failures due to data mismatches.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L243-L248

```diff
        require(
            mintpassIdsToBurn.length == mintPassDnas.length && 
            mintPassDnas.length == fighterTypes.length && 
            fighterTypes.length == modelHashes.length &&
+            fighterTypes.length == iconTypes.length &&
            modelHashes.length == modelTypes.length
        );
```
## [L-02] Fighters should not be allowed to initiate a fight with zero voltage left 
`RankedBattle.updateBattleRecord()` allows for battle initiation even if the fighter's owner lacks the necessary voltage, provided the voltage replenishment time has passed as indicated in the following checks:

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L334-L338

```solidity
        require(
            !initiatorBool ||
            _voltageManagerInstance.ownerVoltageReplenishTime(fighterOwner) <= block.timestamp || 
            _voltageManagerInstance.ownerVoltage(fighterOwner) >= VOLTAGE_COST
        );
```

The delay could possibly lead to dodging point deduction to claim more NRN tokens for the current round and be leveraged in the next round. The loser fighter could also facilitate unstaking NRN tokens during this window to have 0 `curStakeAtRisk` transferred to `_stakeAtRiskAddress` later that I have reported separately. 

A proposed adjustment to the logic strictly enforces that a fighter's owner must have enough voltage upfront if they are initiating a battle, thereby ensuring that all participants meet the same prerequisites for engagement and maintaining the integrity of the battle system.

## [L-03] Enhancing flexibility in permission management 
`GameItems.setAllowedBurningAddresses()` should introduce a second boolean argument to dynamically grant or revoke burning permissions for addresses, streamlining the administrative process and reducing potential risks. This modification allows for more granular control over permissions, enabling administrators to respond swiftly to changing situations, such as addressing errors or security concerns. This approach not only simplifies the contract's interface but also enhances its adaptability and security, making it a robust solution for managing game item transactions in the AI Arena.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L185-L188

```diff
-    function setAllowedBurningAddresses(address newBurningAddress) public {
+    function setAllowedBurningAddresses(address newBurningAddress, bool isAllowed) public {
        require(isAdmin[msg.sender]);
-        allowedBurningAddresses[newBurningAddress] = true;
+        allowedBurningAddresses[newBurningAddress] = isAllowed;
    }
```