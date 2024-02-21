## Low Severity Findings

## [L-01]  Array size mismatch in `MergingPool::getFighterPoints()` returns an invalid array

### Relevant Github Links
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L202-L211

### Details
The function `getFighterPoints()` is designed to retrieve an array of fighter points corresponding to fighter IDs up to a specified maximum ID (maxId). The array size is set to 1 initially, and points are assigned to the first position of the array in each iteration of the loop. The problem is that the points array is not dynamically adjusted based on the input parameter `maxId`.

### Impact
Users will not get an accurate representation of different points corresponding to their different fighters.

### Tools Used
Manual Review

### Recommended Mitigation Steps
Change the size `1` in the points array to `maxId` instead

```diff
-    uint256[] memory points = new uint256[](1);
+    uint256[] memory points = new uint256[](maxId);
```

## [L-02] `RankedBattle::updateBattleRecord()` function updates `totalBattles` variable twice as often as it should per match

### Relevant Github Links
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L348

### Details
This function seems to be called twice by the game server address; once for the initiator and again for the non-initiator in order to update the battle stats of both fighters. If this is the case, then this variable gets updated once more than it should have for every match that occurs, falsely showing the number of total battles. If we look at the official [documentation](https://docs.aiarena.io/gaming-competition/ranking-system) and scroll down to the "Match Ups" section, we can use this information to imply that this function does get called twice; once for the 'Challenger NFT' (initiator) and once for the Opponent from the 'Opponent Pool'

### Impact
`totalBattles` will be showing an incorrect amount of battles that have occurred in all ranked matches, misleading devs and players with false information.

### Tools Used
Manual Review

### Recommended Mitigation Steps
Consider adding a variable in storage `uint8 private totalBattlesIncrementCounter = 0;` that gets incremented whenever `updateBattleRecord()` is called. Once this variable reaches 2, we can increment `totalBattles` and reset `totalBattlesIncrementCounter` back to 0.

In `RankedBattle.sol` storage:
```diff
+    uint8 private totalBattlesIncrementCounter = 0;
```

In `updateBattleRecord()` function:
```diff
function updateBattleRecord() {
     // ... Rest of code
-    totalBattles += 1;
+    totalBattlesIncrementCounter += 1;
+    if(totalBattlesIncrementCounter == 2) {
+        totalBattles += 1;
+        totalBattlesIncrementCounter = 0;
+    }
}
```

## [L-03] Disallowed voltage spenders can still spend voltage in `VoltageManager::spendVoltage()` function 

### Relevant Github Links
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L106

### Details
Even if an admin restricts a user from spending voltage by setting `allowedVoltageSpenders[msg.sender] = false`, the user will still bypass it and be able to spend the voltage anyway due to the first check in the require statement. 

### Impact
Any user can call `spendVoltage` and spend their voltage points even if `allowedVoltageSpenders[msg.sender]` is false. This is either not what the protocol intended or this is a redundant check.

### Tools Used
Manual Review

### Recommended Mitigation Steps
Consider one of two options depending on what the protocol intended it to be:
The natspec specifically states, "This function can be called by the user or an allowed voltage spender". If we go with what the natspec insists, then we can get rid of the `allowedVoltageSpenders[msg.sender]` check since this is redundant:

```diff
-    require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
+    require(spender == msg.sender);
```
Doing this, however, makes the `allowedVoltageSpenders` mapping not used anywhere else in the code, so it seems the protocol did intend this to be used in this function appropriately even though it goes against the natspec comments. If that is the case, then we will adjust it so only allowed voltage spenders are allowed to spend voltage:

```diff
-    function spendVoltage(address spender, uint8 voltageSpent) public {
-        require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
-        if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
-            _replenishVoltage(spender);
-        }
-        ownerVoltage[spender] -= voltageSpent;
-        emit VoltageRemaining(spender, ownerVoltage[spender]);
-    }
```
```diff
+    function spendVoltage(uint8 voltageSpent) public {
+        require(allowedVoltageSpenders[msg.sender], "You are not an allowed voltage spender");
+        if (ownerVoltageReplenishTime[msg.sender] <= block.timestamp) {
+            _replenishVoltage(msg.sender);
+        }
+        ownerVoltage[msg.sender] -= voltageSpent;
+        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
+    }
```

## [L-04] Invalid return type in `FighterOps::viewFighterInfo()` function

### Relevant Github Links
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterOps.sol#L43
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterOps.sol#L79-L104

### Details
The `viewFighterInfo()` function's last parameter returns a uint16 but the last return variable `self.generation` is a uint8.

### Recommended Mitigation Steps
Change the last return parameter to uint8 instead of uint16
```diff
-    returns (address, uint256[6] memory, uint256, uint256, string memory, string memory, uint16)
+    returns (address, uint256[6] memory, uint256, uint256, string memory, string memory, uint8)
```

## [L-05] `FighterFarm::doesTokenExist()` function is never used

### Relevant Github Links
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L299

### Details
The `doesTokensExist()` function is never called anywhere else in the protocol to check if a specific tokenId of a fighter NFT exists.

### Impact
Any function that takes a `tokenId` as a parameter may not actually exist which can lead to unintended behavior of storage variables and events being emitted. 

### Recommended Mitigation Steps
Consider adding a require check for `tokenId` to every function that takes a `tokenId` as a parameter
```solidity
    require(doesTokenExist(tokenId), "Token ID does not exist");
```

## [L-06] `FighterFarm::reRoll()` is not callable due to an error in parameter types

### Relevant Github Links
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L370

### Details
Throughout the entire protocol code base, `tokenId` is of type `uint256` but the `reRoll()` function, which allows a user to re-roll one of their NFT fighters into a new one, takes the `tokenId` as a `uint8`. It's very unlikely a user will be able to successfully call this function given that the tokenId value needs to be within 0-255.

### Impact
Users will not be able to re-roll a new fighter NFT, leaving them stuck with the current fighters they have.

### Recommended Mitigation Steps
Consider changing the type of `tokenId` to a uint256

```diff
-    function reRoll(uint8 tokenId, uint8 fighterType) public {
+    function reRoll(uint256 tokenId, uint8 fighterType) public {
```

## Informational Findings

## [I-01] Incorrect mapping notice in `MergingPool.sol`

### Relevant Github Links 
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L50-L51

### Details
The mapping notice for `mapping(uint256 => address[]) public winnerAddresses;` says "Mapping of address to fighter points" instead of "Mapping of fighter's tokenId to fighter points"

### Recommended Mitigation
Replace the notice with this:
```diff
-     /// @notice Mapping of address to fighter points.
+     /// @notice Mapping of tokenId to fighter points.
```


## [I-02] Official documentation states different `battleResult` values than in source code comments

### Relevant Github Links 
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L411

### Details
The official [documentation](https://docs.aiarena.io/gaming-competition/nrn-rewards) states the following in the "Points Calculation" section:

"Battle Result is either 0 for a loss or tie, and 1 for a win". 

But the code and comments say otherwise:

`/// @param battleResult The result of the battle (0: win, 1: tie, 2: loss).`

### Recommended Mitigation 
Consider changing the documentation to adhere to the source code values.

## [I-03] Official documentation states different voltage usage and replenish values than in source code and comments

### Relevant Github Links
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L52-L53
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L117-L120

### Details
The official [documentation](https://docs.aiarena.io/economic-system/201-economy-design#15909b0656cd431b991607bd4fb9a2cd) states the following in the "In-Game Items" section:

"Each NFT will have 10 Voltage units which are recharged every 24 hours. Each fight in the Ranked Battle arena, expends 1 Voltage unit. This allows for 10 initiated fights a day."

But the code and comments state that each fight expends 10 voltage units, and the recharge replenishes it back to 100 voltage.

### Recommended Mitigation Steps
Consider changing the documentation to adhere to the source code values.


## [I-04] If-statements checking return values for bools should be require statements instead

### Relevant Github Links
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L494
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L283-L289
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L165
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L252
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L81
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L101


### Details
There are several instances throughout the protocol where there is an if-statement `if(success)` to check if `bool success` returns true. Instead of an if-statement, the function should have a require statement to check if `success` returns true or false. If it returns false, the function will revert and save us from mistakenly updating storage values.

### Impact
Storage variables will get updated mistakenly and provide inaccurate information to devs and users if `success` returns false

### Recommended Mitigation Steps
Change every instance of `if(success)` to a require statement:
```solidity
-    if (success) {
+    require(success);
            //rest of code
-       }
```

## Recommendations

## [R-01] Add a check in `VoltageManager::useVoltageBattery()` to make sure that the user doesn't waste using voltage batteries too early

### Relevant Github Links
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L93-L99

### Details
A user can call `useVoltageBattery()` if they have less than 100 voltage. But a user might call the function and replenish their voltage too early, causing them to unnecessarily waste a battery even though they had enough voltage to continue doing ranked battles.

### Recommended Mitigation Steps
Consider adding a check to make sure that the user has 0 voltage before being allowed to use a battery.

```diff
function useVoltageBattery() public {
-    require(ownerVoltage[msg.sender] < 100);
+    require(ownerVoltage[msg.sender] == 0);
     require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
     _gameItemsContractInstance.burn(msg.sender, 0, 1);
     ownerVoltage[msg.sender] = 100;
     emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
}
```

## [R-02] The Game Server may call `RankedBattle::updateBattleRecord()` function passing `mergingPortion = 0` and points may not get merged to `MergingPool.sol`

### Relevant Github Links
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L332
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L448-L451

### Details
When a user wins a ranked match and they do not have any NRN tokens "at risk" in `StakeAtRisk.sol`, the protocol will divert a portion of the points the user gained to the merging pool.
 
There may be a possibility where the `_gameServerAddress` does not send a value greater than 0 into `mergingPortion` inside the `updateBattleRecord()` function. This function will call `_addResultPoints()` and send this variable as an argument. If `mergingPortion = 0`, then the user will not get a portion of their points transferred to `MergingPool.sol`. 

### Recommended Mitigation Steps

Consider adding a check to make sure `mergingPortion > 0`
```diff
-    require(mergingPortion <= 100);
+    require(mergingPortion <= 100 && mergingPortion > 0);
```