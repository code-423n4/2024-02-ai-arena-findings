# [1] `getFighterPoints()` may lead to DoS

**File:** `MergingPool.sol`

[File: src/MergingPool.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L205)
```solidity
205:     function getFighterPoints(uint256 maxId) public view returns(uint256[] memory) {
206:         uint256[] memory points = new uint256[](1);
207:         for (uint256 i = 0; i < maxId; i++) {
208:             points[i] = fighterPoints[i];
209:         }
210:         return points;
211:     }
```
Function `getFighterPoints()` iterates from 0 to `maxId` while `maxId` is provided by user. Whenver `maxId` is too large, the loop would revert with out of gas error - thus it won't be possible to retrieve points for multiple of fighters.
Much better idea would be to re-implement this function and allow to provide both `minId` and `maxId`. Instead of iterating from 0 to `maxId`, it would be more effective to loop from `minId` to `maxId`. That way, when we want to retrieve the points of fighters whose ids are very big, we can specify the starting range (`minId`) which won't revert with out of gas error. E.g., let's say we want to iterate to `maxId = 100 000`. This will be 100 000 iterations which will cost a lot of gas. Instead of iterating from 0, `minId` would allow us to iterate from, e.g., 90 000 to 100 000 - that would be only 10 000 iterations, instead of 100 000.


# [2] Randoms bytes in `_mint()` function are not random at all

**File:** `GameItems.sol`

[File: src/GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L173)
```solidity
173:             _mint(msg.sender, tokenId, quantity, bytes("random"));
174:             emit BoughtItem(msg.sender, tokenId, quantity);
```

While `bytes("random")` suggests that parameter should be random - minting new game items will always pass the same bytes: `random` as the data parameter. Having this might be very misleading.
Even though this data does not need to be random at all - it'd be good practice to have this data different for every new token. It would be better to derive `data` parameter from `tokenId`, instead of having hard-coded string `random`.


# [3] `mint()` in `GameItems.sol` might revert

**File:** `GameItems.sol`

[File: src/GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L158)
```solidity
158:         require(
159:             dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp || 
160:             quantity <= allowanceRemaining[msg.sender][tokenId] 
161:         );
162: 
163:         _neuronInstance.approveSpender(msg.sender, price);
164:         bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, price);
165:         if (success) {
166:             if (dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp) {  
167:                 _replenishDailyAllowance(tokenId);
168:             }
169:             allowanceRemaining[msg.sender][tokenId] -= quantity;
```

When `dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp`, then even when `quantity > allowanceRemaining[msg.sender][tokenId]` - `require` statement at line 158 won't revert (because of the OR operation, only one condition needs to be fulfilled). This implies, that scenario when `quantity > allowanceRemaining[msg.sender][tokenId]` is possible.
Since `dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp`, then, at line 167, we will call function `_replenishDailyAllowance()`. This function sets `allowanceRemaining[msg.sender][tokenId] = allGameItemAttributes[tokenId].dailyAllowance;`. However, nothing guarantees, that `allGameItemAttributes[tokenId].dailyAllowance` needs to be greater than passed variable `quantity`. This - again - proves, that scenario when `quantity > allowanceRemaining[msg.sender][tokenId]` is possible.
Since this scenario is possible - at line 169 - function can revert because of the underflow (we are subtracting bigger value from a smaller one). 

Reverting without any reason (e.g., without clear information provided in the `require`) may be very misleading to the end user. Our recommendation is to add additional `require`, which will make sure that user will know why function might have reverted.

```
require(allowanceRemaining[msg.sender][tokenId] >= quantity, "Trying to mint more than allowed!");
allowanceRemaining[msg.sender][tokenId] -= quantity;
```

Even though this does not constitute any security risk (the revert will occur after all) - it's a good practice to inform a user why the revert occurred.

# [4] Redudant assigment in `getAllowanceRemaining()`

**File:** `GameItems.sol`

[File: src/GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L268)
```solidity
268:     function getAllowanceRemaining(address owner, uint256 tokenId) public view returns (uint256) {
269:         uint256 remaining = allowanceRemaining[owner][tokenId];
270:         if (dailyAllowanceReplenishTime[owner][tokenId] <= block.timestamp) {  
271:             remaining = allGameItemAttributes[tokenId].dailyAllowance;
272:         }
273:         return remaining;
274:     }
```

When `dailyAllowanceReplenishTime[owner][tokenId] <= block.timestamp`, we should immediately return `allGameItemAttributes[tokenId].dailyAllowance`, instead of assigning it to `remaining`. This will shorten the code, remove redundant operations, thus increase the code readability:

```
    function getAllowanceRemaining(address owner, uint256 tokenId) public view returns (uint256) {
        uint256 remaining = allowanceRemaining[owner][tokenId];
        if (dailyAllowanceReplenishTime[owner][tokenId] <= block.timestamp) {  
            return allGameItemAttributes[tokenId].dailyAllowance;
        }
        return remaining;
    }
```

# [5] Using `_approve()` may lead to race condition scenarios

**File:** `Neuron.sol`

According to [Code4rena Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization#approve-race-condition-nc-or-invalid), this issue should be classified as NC.

Will the ongoing discussing continues - if this issue is really a security threat - we've decide to report this potential scenario in the QA report - so that the developer will be able to assess the risk on their own. Nonetheless, we do believe that this issue does not constitute a huge threat - thus should be treated as informational only.

Let's consider a scenario when Alice wants to approve Bob to spend her tokens. She calls `_approve(0xB0b, 1000)` and then she notices, that she mistakenly approved too much. She tries to fix her mistake, by calling `_approve(0xB0b, 100)` again (this time with the proper amount).
However, Bob notices that and front-runs the 2nd `_approve()` by transferring from Alice 1000 tokens - and - after her second `_approve()` - he transfers another 100 tokens. At the end, he managed to get 1100 from Alice (1000 + 100).

To avoid this issue, our recommendation is to use `increaseAllowance`, `decreaseAllowance` functions, which protects from above scenario.

[File: src/Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L184)
```solidity
184:     function approveStaker(address owner, address spender, uint256 amount) public {
185:         require(
186:             hasRole(STAKER_ROLE, msg.sender), 
187:             "ERC20: must have staker role to approve staking"
188:         );
189:         _approve(owner, spender, amount);
190:     }
```

In the above example, user with `STAKER_ROLE`, whenever approving incorrect amount for the first time - may be front-run by the user.

[File: src/Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L171)
```solidity
171:     function approveSpender(address account, uint256 amount) public {
172:         require(
173:             hasRole(SPENDER_ROLE, msg.sender), 
174:             "ERC20: must have spender role to approve spending"
175:         );
176:         _approve(account, msg.sender, amount);
177:     }
```

In the above example, user with `SPENDER_ROLE`, whenever approving incorrect amount for the first time - may be front-run by the user. 

[File: src/Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L127)
```solidity
127:     function setupAirdrop(address[] calldata recipients, uint256[] calldata amounts) external { 
128:         require(isAdmin[msg.sender]);
129:         require(recipients.length == amounts.length);
130:         uint256 recipientsLength = recipients.length;
131:         for (uint32 i = 0; i < recipientsLength; i++) {
132:             _approve(treasuryAddress, recipients[i], amounts[i]);
133:         }
134:     }
```

In the above example, admin, whenever approving incorrect amount for the first time - may be front-run by multiple of users.


# [6] `spendVoltage()` in `VoltageManager.sol` may revert

**File:** `VoltageManager.sol`

[File: src/VoltageManager.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L105)
```solidity
105:     function spendVoltage(address spender, uint8 voltageSpent) public {
106:         require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
107:         if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
108:             _replenishVoltage(spender);
109:         }
110:         ownerVoltage[spender] -= voltageSpent; 
```

In function `spendVoltage()`, there's no additional check which guarantees that ` ownerVoltage[spender] >= voltageSpent`, this implies, that when `voltageSpent` passed by the user is too big - function will revert because of the underflow. While this does not lead to any security risk (function will revert after all) - it's a good practice to inform the end user about the reason of revert.

Reverting without any reason (e.g., without clear information provided in the `require`) may be very misleading to the end user. Our recommendation is to add additional `require`, which will make sure that user will know why function might have reverted:

```
require( ownerVoltage[spender] >= voltageSpent, "Trying to spend too much voltage");
ownerVoltage[spender] -= voltageSpent; 
```

# [7] Incorrect comment in `getUnclaimedRewards()` in `MerginPool.sol`

**File:** `MergingPool.sol`

[File: src/MergingPool.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L169)
```solidity
169:     /// @notice Gets the unclaimed rewards for a specific address.
170:     /// @param claimer The address of the claimer.
171:     /// @return numRewards The amount of unclaimed fighters.
172:     function getUnclaimedRewards(address claimer) external view returns(uint256) {
```

` /// @notice Gets the unclaimed rewards for a specific address.` should be changed to ` /// @notice Gets the amount of unclaimed rewards for a specific address.`, since this function returns only how many unclaimed rewards the address passed as `claimer` parameter has.


# [8] Use descriptive constants instead of hard-coding numbers

**File:** `RankedBattle.sol`

[File: src/RankedBattle.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L439)
```solidity
439:         curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;
```

Above code requires explanation why it divides by `10**4`. Our recommendation is to use constant with descriptive names, instead of hard-coding numbers into the source code.

# [9] Incorrect function name in `MerginPool.sol`

**File:** `MergingPool.sol`

[File: src/MergingPool.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L111)
```solidity
111:     /// @notice Allows the admin to pick the winners for the current round.
[...]
118:     function pickWinner(uint256[] calldata winners) external { 
```

Function `pickWinner()` picks multiple of winners for the current round. This implies, that the function name should be changed to `pickWinners()`. The current function name suggests that only one winner can be chosen.

# [10] Incorrect grammar

**File:** `FighterFarm.sol`

[File: src/FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L186)
```solidity
186:     /// @dev The function verifies the message signature is from the delegated address.
```

`The function verifies the message signature is` should be changed to `The function verifies if the message signature is`


# [11] It's possible to call `updateFighterStaking()` on non-existing `tokenId`

**File:** `FighterFarm.sol`

[File: src/FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L268)
```solidity
268:     function updateFighterStaking(uint256 tokenId, bool stakingStatus) external {  
269:         require(hasStakerRole[msg.sender]);
270:         fighterStaked[tokenId] = stakingStatus;
271:         if (stakingStatus) {
272:             emit Locked(tokenId);
273:         } else {
274:             emit Unlocked(tokenId);
275:         }
276:     }
```

Function `updateFighterStaking()` does not verify if provided `tokenId` exists. This implies, that it would be possible to update `fighterStaked` mapping of future (not yet created) tokens.
Our recommendation is to verify that `tokenId` exists, before updating `fighterStaked[tokenId]`


# [12] It's possibe to set token URI of non-yet-existing tokens

**Files:** `FighterFarm.sol`, `GameItems.sol`

[File: src/FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L180)
```solidity
180:     function setTokenURI(uint256 tokenId, string calldata newTokenURI) external {  
181:         require(msg.sender == _delegatedAddress);
182:         _tokenURIs[tokenId] = newTokenURI;
183:     }
```

[File: src/GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L194)
```solidity
194:     function setTokenURI(uint256 tokenId, string memory _tokenURI) public {
195:         require(isAdmin[msg.sender]);
196:         _tokenURIs[tokenId] = _tokenURI;
197:     }
```

Functions `setTokenURI` does not verify if provided `tokenId` exists. This implies, that it would be possible to update `_tokenURIs` mapping of future (not yet created) tokens.
Our recommendation is to verify that `tokenId` exists, before updating  `_tokenURIs[tokenId]`

# [13] Typos 

**Files:** `FighterFarm.sol`, `RankedBattle.sol`, `AAMintPass.sol`, `GameItems.sol`

[File: src/AAMintPass.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AAMintPass.sol#L84)
```solidity
84:     /// @dev Set the delagated address (Founder-Only)
```

`delagated` should be changed to `delegated`.

[File: src/FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L77)
```solidity
77: 
78:     /// @notice Mapping to keep track of how many times an nft has been re-rolled.
```

`nft` should be changed to `NFT`.

[File: src/RankedBattle.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L415)
```solidity
415:     /// @param fighterOwner The address which owns the fighter whos points are being altered.
```

`whos points` should be changed to `whose points`.

[File: src/GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L57)
```solidity
57:     /// @notice The address that recieves funds of purchased game items.
```

`recieves` should be changed to `receives`.


# [14] Using modifiers instead of hard-coding `require` will improve the code readability

Contract does not implement any modifier. Instead, it hard-codes `require` statements to verify the same information.
E.g., let's consider function `transferOwnership()` from `MergingPool.sol`:

```
    function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
    }
```

It calls `require(msg.sender == _ownerAddress);` to check if `msg.sender` is the owner.
The same line is inside `adjustAdminAccess()` function:

```
   function adjustAdminAccess(address adminAddress, bool access) external {
        require(msg.sender == _ownerAddress);
        isAdmin[adminAddress] = access;
    }  
```

To improve the code quality, it would be much better idea to implement a single modifier, which can be used both for `transferOwnership()` and `adjustAdminAccess()` functions.
This issue occurs across the whole code-base. E.g., in `AAMintPass.sol`, multiple of functions check: `require(msg.sender == founderAddress);`.
To make the QA report clear and easy to read - we decided to report this as an overall issue, without listing all the affected instances.
Our recommendation is to run `grep require  -ri . | grep "msg.sender =="` over the whole code-base (it returns 46 instances) and consider implement modifier for those results.

# [15] Setters should prevent re-setting of the same value

**Files:** `MergingPool.sol`, `RankedBattle.sol`, `Neuron.sol`, `VoltageManager.sol`, `GameItems.sol`

Reported in this QA report instances were missed by the bot-report: `N-63`.

It's possible to call function which updates the state variable with the same value.
It's a good practice to make sure, that whenever we update some value, it's not being set to the same value which it was before the update.
E.g., let's consider a scenario where `isAdmin[0xaaa] = true`. Calling `adjustAdminAccess(0xaaa, true)` is possible, even though it won't change the value of the variable: `isAdmin[0xaaa]` will remain `true`.
Our recommendation is to implement additional check which verifies if value is really changed: `require(isAdmin[adminAddress] != access, "Nothing to change")`.

[File: src/MergingPool.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L098)
```solidity
098:     function adjustAdminAccess(address adminAddress, bool access) external {
099:         require(msg.sender == _ownerAddress);
100:         isAdmin[adminAddress] = access;
101:     }   
```

[File: src/RankedBattle.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L176)
```solidity
176:     function adjustAdminAccess(address adminAddress, bool access) external {
177:         require(msg.sender == _ownerAddress);
178:         isAdmin[adminAddress] = access;
179:     }  
```

[File: src/GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L117)
```solidity
117:     function adjustAdminAccess(address adminAddress, bool access) external {
118:         require(msg.sender == _ownerAddress);
119:         isAdmin[adminAddress] = access;
120:     }  
```

[File: src/Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L118)
```solidity
118:     function adjustAdminAccess(address adminAddress, bool access) external {
119:         require(msg.sender == _ownerAddress);
120:         isAdmin[adminAddress] = access;
121:     }  
```

[File: src/VoltageManager.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L73)
```solidity
73:     function adjustAdminAccess(address adminAddress, bool access) external { 
74:         require(msg.sender == _ownerAddress);
75:         isAdmin[adminAddress] = access;
76:     }  
```

[File: src/VoltageManager.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L82)
```solidity
82:     function adjustAllowedVoltageSpenders(address allowedVoltageSpender, bool allowed) external {
83:         require(isAdmin[msg.sender]);
84:         allowedVoltageSpenders[allowedVoltageSpender] = allowed;
85:     }
```

# [16] Important, state-changing functions do not emit events

**Files:** `FighterFarm.sol`, `MergingPool.sol`, `AiArenaHelper.sol`, `RankedBattle.sol`, `Neuron.sol`, `AAMintPass.sol`, `VoltageManager.sol`, `GameItems.sol`

Code-base implements multiple of functions which change the state of the contract (e.g. transfer ownership, adds/removes new admins, changing the addresses, etc.). None of these functions emit events.
Whenever contract changes the state of the blockchain and perform important/critical operation (e.g. transferring ownership is critical operation) - it should emit an event.

The QA report contains the list of functions which do not emit events

[File: src/FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L120)
```solidity
120:     function transferOwnership(address newOwnerAddress) external {
129:     function incrementGeneration(uint8 fighterType) external returns (uint8) {
139:     function addStaker(address newStaker) external {
147:     function instantiateAIArenaHelperContract(address aiArenaHelperAddress) external {
155:     function instantiateMintpassContract(address mintpassAddress) external {
163:     function instantiateNeuronContract(address neuronAddress) external {
171:     function setMergingPoolAddress(address mergingPoolAddress) external {
180:     function setTokenURI(uint256 tokenId, string calldata newTokenURI) external {  
283:     function updateModel(
```

[File: src/MergingPool.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L89)
```solidity
89:     function transferOwnership(address newOwnerAddress) external {
98:     function adjustAdminAccess(address adminAddress, bool access) external {
106:     function updateWinnersPerPeriod(uint256 newWinnersPerPeriodAmount) external {
```   


[File: src/AAMintPass.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AAMintPass.sol#L55)
```solidity
55:     function transferOwnership(address _newFounderAddress) external {
65:     function addAdmin(address _newAdmin) external {
72:     function removeAdmin(address adminAddress) external {
79:     function setFighterFarmAddress(address _fighterFarmAddress) external {
86:     function setDelegatedAddress(address _delegatedAddress) external {
93:     function setPaused(bool _state) external {
```


[File: src/RankedBattle.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L167)
```solidity
167:     function transferOwnership(address newOwnerAddress) external {
176:     function adjustAdminAccess(address adminAddress, bool access) external {
184:     function setGameServerAddress(address gameServerAddress) external {
192:     function setStakeAtRiskAddress(address stakeAtRiskAddress) external {
218:     function setRankedNrnDistribution(uint256 newDistribution) external {
226:     function setBpsLostPerLoss(uint256 bpsLostPerLoss_) external {
233:     function setNewRound() external {
```


[File: src/GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L108)
```solidity
108:     function transferOwnership(address newOwnerAddress) external {
117:     function adjustAdminAccess(address adminAddress, bool access) external {
185:     function setAllowedBurningAddresses(address newBurningAddress) public {
194:     function setTokenURI(uint256 tokenId, string memory _tokenURI) public {
```


[File: src/Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L118)
```solidity
118:     function adjustAdminAccess(address adminAddress, bool access) external {
127:     function setupAirdrop(address[] calldata recipients, uint256[] calldata amounts) external {
```

[File: src/AiArenaHelper.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L61)
```solidity
61:     function transferOwnership(address newOwnerAddress) external {
```


[File: src/VoltageManager.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L64)
```solidity
64:     function transferOwnership(address newOwnerAddress) external {
73:     function adjustAdminAccess(address adminAddress, bool access) external { 
82:     function adjustAllowedVoltageSpenders(address allowedVoltageSpender, bool allowed) external {
```
