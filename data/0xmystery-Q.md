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
## [L-04] Navigating the shift by ensuring fair transaction ordering in a decentralized Arbitrum
As `Arbitrum` considers moving towards a more decentralized sequencer model, the platform faces the challenge of maintaining its current mitigation of frontrunning risks inherent in a "first come, first served" system. The transition could reintroduce vulnerabilities to transaction ordering manipulation, demanding innovative solutions to uphold transaction fairness. Strategies such as commit-reveal schemes, submarine sends, Fair Sequencing Services (FSS), decentralized MEV mitigation techniques, and the incorporation of time-locks and randomness could play pivotal roles. These measures aim to preserve the integrity of transaction sequencing, ensuring that Arbitrum's evolution towards decentralization enhances its ecosystem without compromising the security and fairness that are crucial for user trust and platform reliability.

## [L-05] Accelerated token minting and sustainability concerns in DAO-governed protocols
With the Neuron token contract initially designed for a modest minting schedule of [5,000 tokens](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L157) twice a week, an escalated minting proposal to 5 million tokens bi-weekly by a DAO-governed consensus (presuming the protocol is headed to that direction) raises significant sustainability questions. Such an aggressive inflation rate would deplete the remaining 300 million mintable supply:

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L36-L43

```solidity
    MAX_SUPPLY -= (INITIAL_TREASURY_MINT + INITIAL_CONTRIBUTOR_MINT); 
```
within approximately 30 weeks or about 6.9 months, starkly contrasting the centuries-spanning timeline under the original minting pace. This rapid approach to reward distribution underscores the critical need for careful consideration of tokenomics and long-term viability in DAO governance decisions, especially when balancing incentive mechanisms against the risk of premature token supply exhaustion.