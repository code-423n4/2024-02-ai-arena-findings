## [QA/Low-01] Redundant setting of `attributeProbabilities`

### Code references

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L49

### Description
In the `constructor` of `AiArenaHelper.sol`, the `addAttributeProbabilities(0, probabilities);` function is called. The function `addAttributeProbabilities()` updates `attributeProbabilities`. Next, the constructor sets the `attributeProbabilities` values in the same way.

```solidity
constructor(uint8[][] memory probabilities) {
    ...
    ...
    // Initialize the probabilities for each attribute
    addAttributeProbabilities(0, probabilities);

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
        attributeProbabilities[0][attributes[i]] = probabilities[i];
        ...
        ...
    }
    ...
    ...
}

function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
    ...
    ...
    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
        attributeProbabilities[generation][attributes[i]] = probabilities[i];
    }
    ...
    ...
}
```

This is redundant, and can be removed from the `constructor` for loop.

### Recommendation

Remove the `attributeProbabilities` variable update from the constructor.

```diff
constructor(uint8[][] memory probabilities) {
    _ownerAddress = msg.sender;

    // Initialize the probabilities for each attribute
    addAttributeProbabilities(0, probabilities);

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
-       attributeProbabilities[0][attributes[i]] = probabilities[i];
        attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
    }
}
```

---

## [QA/Low-02] Update `probabilities.length` check to be against `attributes.length` and not against magic numbers

### Code references

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L133

### Description

In `AiArenaHelper::addAttributeProbabilities()`, there is a check that validates the length of `probabilities` to be equal to `6`. The number `6` comes from the length of attributes. As the length of attributes can change as contract updates in the future, its easy to miss this magic number check.

```solidity
function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
    ...
    ...
    require(probabilities.length == 6, "Invalid number of attribute arrays");
    ...
    ...
}
```

### Recommendation

Update the check to be against the length of the `attributes` instead:

```diff
function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
    require(msg.sender == _ownerAddress);
-   require(probabilities.length == 6, "Invalid number of attribute arrays");
+   require(probabilities.length == attributes.length, "Invalid number of attribute arrays");


    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
        attributeProbabilities[generation][attributes[i]] = probabilities[i];
    }
}
```

Similar checks are done in `AiArenaHelper::addAttributeDivisor()` already.

---

## [QA/Low-03] Add validation to increment generation function in Fighter Farm contract

### Code references

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L129

### Description

The NatSpec of the `FighterFarm::incrementGeneration()` mentions that the `fighterType` function input can only be 0 or 1, but there are no checks that enforce this. As this is an external function which will be called by other contracts, it assumes that such a check will be enforced by the calling contracts, which will not always be true, and makes the function prone to errors.

```solidity
...
...
/// @param fighterType Type of fighter either 0 or 1 (champion or dendroid).
...
...
function incrementGeneration(uint8 fighterType) external returns (uint8) {
    ...
    ...
}
```

### Recommendation

Add a check in the function to ensure that `fighterType` is always 0 or 1.

```diff
/// @notice Increase the generation of the specified fighter type.
/// @dev Only the owner address is authorized to call this function.
/// @param fighterType Type of fighter either 0 or 1 (champion or dendroid).
/// @return Generation count of the fighter type.
function incrementGeneration(uint8 fighterType) external returns (uint8) {
    require(msg.sender == _ownerAddress);
+   require(fighterType <= 1);
    generation[fighterType] += 1;
    maxRerollsAllowed[fighterType] += 1;
    return generation[fighterType];
}
```

---

## [QA/Low-04] Redundant fetching of stake at risk

### Code references

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L341

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L430

### Description

The `RankedBattle::updateBattleRecord()` function fetches the stake at risk by calling the function `StakeAtRisk.getStakeAtRisk()` function. Then it calls an internal function `RankedBattle::_addResultPoints()` which again fetches the stake at risk by calling the function `StakeAtRisk.getStakeAtRisk()` function. This is redundant, and the second call can be eliminated.

The contracts have the following function to transfer the ownership:

```solidity
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
    ...
    uint256 stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
    if (amountStaked[tokenId] + stakeAtRisk > 0) {
        _addResultPoints(battleResult, tokenId, eloFactor, mergingPortion, fighterOwner);
    }
    ...
    ...
}

function _addResultPoints(
    uint8 battleResult, 
    uint256 tokenId, 
    uint256 eloFactor, 
    uint256 mergingPortion,
    address fighterOwner
) 
    private 
{
    ...
    ...
    stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
    ...
    ...
}
```

### Recommendation
Remove the call to `StakeAtRisk.getStakeAtRisk()` in the internal function `RankedBattle::_addResultPoints()`, and instead pass the value of stake at risk to `RankedBattle::_addResultPoints()` from `RankedBattle::updateBattleRecord()`.

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
    ...
    uint256 stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
    if (amountStaked[tokenId] + stakeAtRisk > 0) {
-       _addResultPoints(battleResult, tokenId, eloFactor, mergingPortion, fighterOwner);
+       _addResultPoints(battleResult, tokenId, eloFactor, mergingPortion, stakeAtRisk, fighterOwner);
    }
    ...
    ...
}

function _addResultPoints(
    uint8 battleResult, 
    uint256 tokenId, 
    uint256 eloFactor, 
    uint256 mergingPortion,
+   uint256 stakeAtRisk,
    address fighterOwner
) 
    private 
{
-   uint256 stakeAtRisk;
    uint256 curStakeAtRisk;
    uint256 points = 0;

    /// Check how many NRNs the fighter has at risk
-   stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
    ...
    ...
}
```

---

## [QA/Low-05] Use 2-step transfer ownership

### Code references
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L61

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L64

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L85

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L89

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L108

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L120

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L167

### Description

The contracts have the following function to transfer the ownership:

```solidity
function transferOwnership(address newOwnerAddress) external {
    require(msg.sender == _ownerAddress);
    _ownerAddress = newOwnerAddress;
}
```

Even though the owners are trusted and are assumed to "never" commit mistakes, its just something that is inevitable. Such mistakes can lead to loss of owner protected functions of the contracts.

### Recommendation
To protect the functioning of the protocol from such inevitable mistakes of the "trusted" owners, it is recommended to use Ownable2Step from OpenZeppelin - https://docs.openzeppelin.com/contracts/5.x/api/access#Ownable2Step

---