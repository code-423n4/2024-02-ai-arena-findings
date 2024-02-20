# Gas Optimizations

**Note : _G-08_ and _G-09_ contains only those instances which were missed by bot. Since they are major gas savings so I included those missed instances**
| Number | Issue |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------- |
| [[G-01](#g-01-pack-structs-by-reducing-variable-sizes-to-save-storage-slots-saves-12000-gas)] | Pack structs by reducing variable sizes to save storage slots (saves ~12000 Gas) |
| [[G-02](#g-02-state-variables-can-be-packed-into-fewer-storage-slot-saves-8000-gas)] | State variables can be packed into fewer storage slot (saves ~8000 Gas) |
| [[G-03](#g-03-restructure-nftsclaimed-mapping-elements-and-make-struct-elementsaves-2000-gas)] | Restructure `nftsClaimed` mapping elements and make struct element(saves ~2000 Gas) |
| [[G-04](#g-04-remove-unnecessary-isselectioncomplete-mapping-from-mergingpool-contractsaves-22100-gas)] |Remove unnecessary `isSelectionComplete` mapping from `MergingPool` contract(saves ~22100 Gas) |
| [[G-05](#g-05-remove-unnecessary-addattributeprobabilities-function-call-in-constructor)] | Remove unnecessary `addAttributeProbabilities` function call in `constructor` |
| [[G-06](#g-06-refactor-code-to-save-gassaves-100-gas)] | `Refactor` code to save gas(saves ~100 Gas) |
| [[G-07](#g-07-external-calls-should-be-cached-instead-of-re-calling-the-same-function-in-the-same-transaction)] | `External` calls should be `cached` instead of re-calling the same `function` in the same transaction |
| [[G-08](#g-08-dont-read-state-variable-in-loopinstances-missed-by-bot-report)] | Don't read `state` variable in `loop`(_Instances missed by bot report_) |
| [[G-09](#g-09-update-state-variable-outside-of-the-loop-after-accumulating-results-in-stack-variableinstances-missed-by-bot-report)] | Update `state` variable outside of the loop after `accumulating` results in `stack` variable(_Instances missed by bot report_) |
| [[G-10](#g-10-remove-unnecessary-function-parameter)] | Remove unnecessary function parameter |
| [[G-11](#g-11-no-need-to-inherit-same-contract-again-in-child-contract-if-it-is-already-inherited-in-parent)] | No need to `inherit` same contract again in child contract if it is already inherited in parent |
| [[G-12](#g-11-no-need-to-inherit-same-contract-again-in-child-contract-if-it-is-already-inherited-in-parent)] | Refactor `return` statement to use `short-circuiting` can save some Gcoldsloads most of the times.|

## [G-01] Pack structs by reducing variable sizes to save storage slots (saves ~12000 Gas)

### `SAVE: 12000 GAS, 6 SLOT`

As the solidity EVM works with 32 bytes variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot this saves gas when writing to storage ~20000 gas. If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper.

### `head`, `eyes`, `mouth`, `body`, `hands` and `feet` can be packed into single storage slot `Gas Saved: 10000 GAS, 5 SLOTs`.

Since we can see in [AIArenaHelper::createPhysicalAttributes](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L83-L111) function when fighter attributes are created using FighterPhysicalAttributes struct. There are 3 possible conditions, 1st when `dendroidBool` is true then all attributes will set to 99.
2nd if dendroidBool is false and `else` condition triggers. Then inside loop there are if and else block. In `if` block inside `for` loop all attributes are set to 50.
3rd if `else` condition triggers inside `for` loop then attribute values will be made from [dnaToIndex](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L169C5-L186C6) function which contains a `for` loop that loop iterator `i` is uint8 type that means `i` can be max 255 and dnaToIndex returns `i+1`.That means `dnaToIndex` can return max. 256 only. Which is directly assigned into as attributes in `createPhysicalAttributes` function `else` block inside `for` loop. That shows max value of attributes can be max to max 256 can't go beyond that.

Also `createPhysicalAttributes` called in [FighterFarm::\_createNewFighter](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L484C5-L531C6) function these attributes are assigned in fighters array. After creating these attributes they are never updated.

By considering all these conditions we can say that `uint16` is more than enough to hold any physical attribute value.
Which can save `5 SLOTS` in storage for every fighter.

```solidity
File : src/FighterOps.sol

26:  struct FighterPhysicalAttributes {
27:        uint256 head;
28:        uint256 eyes;
29:        uint256 mouth;
30:        uint256 body;
31:        uint256 hands;
32:        uint256 feet;
33:    }

```

[FighterOps.sol#L26C5-L33C6](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L26C5-L33C6)

**Recommended Mitigation Steps:**

```diff
File : src/FighterOps.sol

26:  struct FighterPhysicalAttributes {
-27:        uint256 head;
-28:        uint256 eyes;
-29:        uint256 mouth;
-30:        uint256 body;
-31:        uint256 hands;
-32:        uint256 feet;

+27:        uint16 head;
+28:        uint16 eyes;
+29:        uint16 mouth;
+30:        uint16 body;
+31:        uint16 hands;
+32:        uint16 feet;
33:    }

```

### `dailyAllowance` and `transferable` can be packed in single storage slot `SAVES: 2000 GAS, 1 SLOT`

`dailyAllowance` can easily reduced to `uint16` since in [GameItems::createGameItem](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L208C5-L229C11) function the input parameter type for `dailyAllowance` is uint16. And after assigning it this variable never updated. So `uint16` is more than sufficient to hold it.

[GameItems::createGameItem](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L208C5-L229C11) function

```solidity
File : src/GameItems.sol

208:    function createGameItem(
   ...
215:        uint16 dailyAllowance
```

[GameItems.sol#L208-L215](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L208-L215)

So we can easily use uint16 for `dailyAllowance` and pack with boolean variable `transferable`

```solidity
File : src/GameItems.sol

35:   struct GameItemAttributes {
36:        string name;
37:        bool finiteSupply;
38:        bool transferable;
39:        uint256 itemsRemaining;
40:        uint256 itemPrice;
41:        uint256 dailyAllowance;
42:    }
```

[GameItems.sol#L35C2-L42C6](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L35C2-L42C6)

**Recommended Mitigation Steps:**

```diff
File : src/GameItems.sol

35:   struct GameItemAttributes {
36:        string name;
37:        bool finiteSupply;
38:        bool transferable;
+          uint16 dailyAllowance;
39:        uint256 itemsRemaining;
40:        uint256 itemPrice;
-41:        uint256 dailyAllowance;
42:    }

```

## [G-02] State variables can be packed into fewer storage slot (saves ~8000 Gas)

### `SAVE: 8000, 4 SLOT`

### NOTE: Not in bot report

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper.

### `roundId`, `bpsLostPerLoss` and `_stakeAtRiskAddress` can be packed in single slot `SAVES: 4000 GAS, 2 SLOT`

`roundId` only increase when new rounds starts. Typically new round starts in 1-2 weeks. Even if newRound starts 1000 times every second still `uint64` is sufficient to hold `roundId`. So we can safely reduce the size of `roundId` to `uint64` to save 1 storage slot.

`bpsLostPerLoss` holding basis points which max value can only 10,000 so `uint32` is more than sufficient to hold `bpsLostPerLoss` so we can safely reduce the size to `uin32` to save 1 storage slot.

```solidity
File : src/RankedBattle.sol

62:    uint256 public roundId = 0;

66:    uint256 public bpsLostPerLoss = 10;

69:    address _stakeAtRiskAddress;

```

[RankedBattle.sol#L62C1-L69C33](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L62C1-L69C33)

**Recommended Mitigation Steps:**

```diff
File : src/RankedBattle.sol

-62:    uint256 public roundId = 0;
+62:    uint64 public roundId = 0;

-66:    uint256 public bpsLostPerLoss = 10;
+66:    uint32 public bpsLostPerLoss = 10;

69:    address _stakeAtRiskAddress;

```

### `roundId` and `treasuryAddress` can be packed in single slot `SAVES: 2000 GAS, 1 SLOT`

Similarly like above `roundId` size can also be reduced here to `uin64` saves 1 slot.

```solidity
File : src/StakeAtRisk.sol

27:    uint256 public roundId = 0;

30:    address public treasuryAddress;
```

[StakeAtRisk.sol#L27-L30](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L27C1-L30C36)

**Recommended Mitigation Steps:**

```diff
File : src/StakeAtRisk.sol

-27:    uint256 public roundId = 0;
+27:    uint64 public roundId = 0;

30:    address public treasuryAddress;
```

### `roundId` and `_ownerAddress` can be packed in single slot `SAVES: 2000 GAS, 1 SLOT`

Similarly like above `roundId` size can also be reduced here to `uin64` saves 1 slot.

```solidity
File : src/MergingPool.sol

29:  uint256 public roundId = 0;

35:  address _ownerAddress;
```

[MergingPool.sol#L29-L35](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L29C1-L35C27)

**Recommended Mitigation Steps:**

```diff
File : src/MergingPool.sol

-29:  uint256 public roundId = 0;
+29:  uint64 public roundId = 0;

35:  address _ownerAddress;
```

## [G-03] Restructure `nftsClaimed` mapping elements and make struct element(saves ~2000 Gas)

The current implementation of the `nftsClaimed` mapping in `FighterFarm` employs `uint8` keys `0` and `1` to represent distinct values. This approach may lead to suboptimal gas consumption.
Instead of inner mapping since it has only two keys 0 and 1 based on fighterType. 0 for dendroid and 1 for champion fighter types

### SAVES 1 SLOT, 2000 GAS on each address key

```solidity
File : src/FighterFarm.sol

88: mapping(address => mapping(uint8 => uint8)) public nftsClaimed;

        ...
191:  function claimFighters(
        ...

199:        bytes32 msgHash = bytes32(keccak256(abi.encode(
200:            msg.sender,
201:            numToMint[0],
202:            numToMint[1],
203:            nftsClaimed[msg.sender][0],
204:            nftsClaimed[msg.sender][1]
205:        )));
206:        require(Verification.verify(msgHash, signature, _delegatedAddress));
207:        uint16 totalToMint = uint16(numToMint[0] + numToMint[1]);
208:        require(modelHashes.length == totalToMint && modelTypes.length == totalToMint);
209:        nftsClaimed[msg.sender][0] += numToMint[0];
210:        nftsClaimed[msg.sender][1] += numToMint[1];

```

[FighterFarm.sol#L88](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L88) , [FighterFarm.sol#L199C1-L210C52](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L199C1-L210C52)

**Recommended Mitigation Steps:**

Consider restructuring the nftsClaimed mapping by introducing a struct to improve gas efficiency. This optimization involves consolidating related data within a single slot for each address key, potentially reducing gas costs in Ethereum transactions.
`value1` and `value2` can come in 1 slot. It can save 1 SLOT every time for each address key for outer mapping. Since mapping value always take at least 1 slot and no other can be packed there but in struct 2 values can fit together if their sizes are less than 32 bytes.

```diff
File : src/FighterFarm.sol

-88: mapping(address => mapping(uint8 => uint8)) public nftsClaimed;
+88: mapping(address => NFTClaim) public nftsClaimed;

+ struct NFTClaim {
+    uint8 value1;
+    uint8 value2;
+ }

```

## [G-04] Remove unnecessary `isSelectionComplete` mapping from `MergingPool` contract(saves ~22100 Gas)

`isSelectionComplete` mapping only used in pickWinner function.
The `isSelectionComplete` mapping is flagged as redundant as its value only becomes true when `roundId` increases. By removing this mapping you can save `2 warm loads` and `20,000 gas` associated with transitioning from `false` to `true`. External validation can be achieved by checking the current `roundId` without the need for this mapping. For all the roundIds less than current selection is completed. Since after completing selection roundId increases here. Only here increase so every new roundID this mapping will be false. and roundId is read from storage so `isSelectionComplete[roundId]` will always read for current roundId here. And it is not decreased after that ever. So it is totally safe to remove `isSelectionComplete` mapping from here.

### GAS SAVED 22100 GAS (1 SSTORE and 1 Gcoldsload)

`false` to `true` takes 20000 gas. 1 Gcoldsload takes 2100 Gas

```solidity
File : src/MergingPool.sol

57:   mapping(uint256 => bool) public isSelectionComplete;

118: function pickWinner(uint256[] calldata winners) external {
        ...
121:        require(!isSelectionComplete[roundId], "Winners are already selected");
        ...
130:        isSelectionComplete[roundId] = true;
131:        roundId += 1;
132:    }

```

[MergingPool.sol#L118-L132](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L118C2-L132C6)

**Recommended Mitigation Steps:**

The `isSelectionComplete` mapping in the `MergingPool` contract appears redundant and consumes unnecessary gas. It is suggested to remove this mapping as it serves no purpose beyond its current context.

```diff
File : src/MergingPool.sol

-57:   mapping(uint256 => bool) public isSelectionComplete;

118: function pickWinner(uint256[] calldata winners) external {
        ...
-121:        require(!isSelectionComplete[roundId], "Winners are already selected");
        ...
-130:        isSelectionComplete[roundId] = true;
131:        roundId += 1;
132:    }

```

## [G-05] Remove unnecessary `addAttributeProbabilities` function call in `constructor`

Since the for loop and `addAttributeProbabilities` function are duplicating the same task except providing one extra check for the probability length. Consider removing the function call and also add an extra check for the probability length before performing the array assignment directly in the loop.

### SAVES 2 SSTORE every iteration

```solidity
File : src/AiArenaHelper.sol

41: constructor(uint8[][] memory probabilities) {
42:        _ownerAddress = msg.sender;
43:
44:        // Initialize the probabilities for each attribute
45:        addAttributeProbabilities(0, probabilities);
46:
47:        uint256 attributesLength = attributes.length;
48:        for (uint8 i = 0; i < attributesLength; i++) {
49:            attributeProbabilities[0][attributes[i]] = probabilities[i];
50:            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
51:        }
52:    }

...

131: function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
132:        require(msg.sender == _ownerAddress);
133:        require(probabilities.length == 6, "Invalid number of attribute arrays");
134:
135:        uint256 attributesLength = attributes.length;
136:        for (uint8 i = 0; i < attributesLength; i++) {
137:            attributeProbabilities[generation][attributes[i]] = probabilities[i];
138:        }
138:    }

```

[AiArenaHelper.sol#L41-L52](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L41-L52) , [AiArenaHelper.sol#L131-L139](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131-L139)

**Recommended Mitigation Steps:**

```diff
File : src/AiArenaHelper.sol

41: constructor(uint8[][] memory probabilities) {
42:        _ownerAddress = msg.sender;
43:
44:        // Initialize the probabilities for each attribute
-45:        addAttributeProbabilities(0, probabilities);
+         require(probabilities.length == 6, "Invalid number of attribute arrays");
46:
47:        uint256 attributesLength = attributes.length;
48:        for (uint8 i = 0; i < attributesLength; i++) {
49:            attributeProbabilities[0][attributes[i]] = probabilities[i];
50:            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
51:        }
52:    }
```

## [G-06] `Refactor` code to save gas(saves ~100 Gas)

### Refactor `createPhysicalAttributes` function to 1 SLOAD

We can save 1 SLOAD refactoring code by caching `attributes.length` earlier.

```solidity
File : src/AiArenaHelper.sol

95:  } else {
96:        uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributes.length);
97:
98:        uint256 attributesLength = attributes.length;
99:        for (uint8 i = 0; i < attributesLength; i++) {

```

[AiArenaHelper.sol#L95C8-L99C59](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L95C8-L99C59)

**Recommended Mitigation Steps:**

```diff
File : src/AiArenaHelper.sol

95:  } else {
+98:        uint256 attributesLength = attributes.length;
96:        uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributes.length);
97:
-98:        uint256 attributesLength = attributes.length;
99:        for (uint8 i = 0; i < attributesLength; i++) {

```

## [G-07] `External` calls should be `cached` instead of re-calling the same `function` in the same transaction

### Cache `allowance(account, msg.sender)` `SAVES: 1 External call`

```solidity
File : src/Neuron.sol

196: function burnFrom(address account, uint256 amount) public virtual {
197:        require(
198:            allowance(account, msg.sender) >= amount,
199:            "ERC20: burn amount exceeds allowance"
200:        );
201:        uint256 decreasedAllowance = allowance(account, msg.sender) - amount;

```

[Neuron.sol#L196-L201](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L196-L201)

**Recommended Mitigation Steps:**

```diff
File : src/Neuron.sol

196: function burnFrom(address account, uint256 amount) public virtual {
+        uint256 _allowance = allowance(account, msg.sender);
197:        require(
-198:            allowance(account, msg.sender) >= amount,
+198:            _allowance >= amount,
199:            "ERC20: burn amount exceeds allowance"
200:        );
-201:        uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
+201:        uint256 decreasedAllowance = _allowance - amount;

```

## [G-08] Don't read `state` variable in `loop`(_Instances missed by bot report_)

Reading state variables in a loop within a smart contract can incur significant gas costs. Each read operation involves interacting with the blockchain, leading to increased computational complexity and higher gas consumption, particularly in loops with numerous iterations.

**Note: Since these instances missed by bot in this fining and contains major gas saving so I reported them here.**

### Total GAS SAVED overall 3 SLOAD each iteration in all 3 instances

### Instance#1

`roundId` can be cached saves 1 SLOAD on each iteration(~100 Gas)

```solidity
File : src/MergingPool.sol

149: for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {

```

[MergingPool.sol#L149](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L149)

**Recommended Mitigation Steps:**

```diff
File : src/MergingPool.sol

+     uint256 _roundId = roundId;

-149: for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
+149: for (uint32 currentRound = lowerBound; currentRound < _roundId; currentRound++) {

```

### Instance#2

`roundId` can be cached saves 1 SLOAD on each iteration(~100 Gas)

```solidity
File : src/MergingPool.sol

176:  for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {

```

[MergingPool.sol#L176](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L176)

**Recommended Mitigation Steps:**

```diff
File : src/MergingPool.sol

+     uint256 _roundId = roundId;

-176: for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
+176: for (uint32 currentRound = lowerBound; currentRound < _roundId; currentRound++) {

```

## [G-09] Update `state` variable outside of the loop after `accumulating` results in `stack` variable(_Instances missed by bot report_)

To enhance efficiency it is recommended to move the update of the state variable outside the loop after accumulating results in a temporary stack variable. This can help reduce gas costs by minimizing state variable updates within the loop.

**Note: Since these instances missed by bot in this fining and contains major gas saving so I reported them here.**

### GAS SAVED Saves 1 SSTORE and 1 SLOAD on each iteration

`numRoundsClaimed[msg.sender]` can be updated outside the loop one time since it doesn't depend on loop iterator `i`.

```solidity
File : src/MergingPool.sol

148:  uint32 lowerBound = numRoundsClaimed[msg.sender];
149:   for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
150:       numRoundsClaimed[msg.sender] += 1;
           ...
               }

```

[MergingPool.sol#L148-L150](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L148-L150)

**Recommended Mitigation Steps:**

```diff
File : src/MergingPool.sol


+   uint32 c_numClaimed = numRoundsClaimed[msg.sender];

149:   for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
-150:       numRoundsClaimed[msg.sender] += 1;
+150:      c_numClaimed += 1;
           ...
            }
+        numRoundsClaimed[msg.sender] = c_numClaimed;
```

## [G-10] Remove unnecessary function parameter

The `owner` address is passed as a `parameter` and immediately `returned` without any meaningful use within the `function`. It seems `redundant`, as the function could `utilize` the address where it is `called`, eliminating the need for this `additional` parameter. Align the expectations with the actual usage to enhance code clarity and efficiency.

```solidity
File : src/FighterOps.sol

79  function viewFighterInfo(
80          Fighter storage self,
81          address owner
82      )
83          public
84          view
85          returns (
86              address,
...
94      {
95          return (
96              owner,
...
105 }

```

[FighterOps.sol#L79C5-L105C2](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L79C5-L105C2)

**Recommended Mitigation Steps:**

```diff
File : src/FighterOps.sol

79  function viewFighterInfo(
80          Fighter storage self,
-81          address owner
82      )
83          public
84          view
85          returns (
-86              address,
...
94      {
95          return (
-96              owner,
...
104     }
105 }

```

## [G-11] No need to `inherit` same contract again in child contract if it is already inherited in parent

Since `ERC721Enumerable` already inherits `ERC721`. So it is redundant to inherit this again in child contract if it is already inherited by parent `ERC721Enumerable`.

```solidity
File : src/FighterFarm.sol

abstract contract ERC721Enumerable is ERC721, IERC721Enumerable {

```

Since `ERC721Enumerable` already inherits `ERC721`. So no need to inherit again in `FighterFarm` contract.

```solidity
File : src/FighterFarm.sol

16: contract FighterFarm is ERC721, ERC721Enumerable {

```

[FighterFarm.sol#L16](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L16)

**Recommended Mitigation Steps:**

```diff
File : src/FighterFarm.sol

-16: contract FighterFarm is ERC721, ERC721Enumerable {
+16: contract FighterFarm is ERC721Enumerable {

```

## [G-12] Refactor `return` statement to use `short-circuiting` can save some Gcoldsloads most of the times.

Place less gas consuming first and then more gas consuming since if the less gas consuming became false it is not going to check the next condition. In this case `!fighterStaked[tokenId]` consume less gas then `_isApprovedOrOwner(msg.sender, tokenId)`.

`_isApprovedOrOwner(msg.sender, tokenId)` take more gas than others since it contains 4 SLOADs and multiple function calls inside it. Whereas `fighterStaked[tokenId]` takes only 1 SLOAD. And `balanceOf(to) < MAX_FIGHTERS_ALLOWED` takes 1 SLOAD and one function call. So we can change their order in return statement from less gas consuming to more gas consuming.

### Gas Saved some Gcoldsloads most of the times

```solidity
File : src/FighterFarm.sol

540:  return (
541:          _isApprovedOrOwner(msg.sender, tokenId) &&
542:          balanceOf(to) < MAX_FIGHTERS_ALLOWED &&
543:          !fighterStaked[tokenId]
544:        );
```

[FighterFarm.sol#L540C8-L544C11](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L540C8-L544C11)

**Recommended Mitigation Steps:**

```diff
File : src/FighterFarm.sol

540:  return (
-541:          _isApprovedOrOwner(msg.sender, tokenId) &&
+543:          !fighterStaked[tokenId]
542:          balanceOf(to) < MAX_FIGHTERS_ALLOWED &&
+541:          _isApprovedOrOwner(msg.sender, tokenId) &&
-543:          !fighterStaked[tokenId]
544:        );
```
