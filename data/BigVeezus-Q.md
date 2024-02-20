
## [L-1] `RankedBattle::_addResultPoints` function does not update the `globalStakedAmount` after an amount has been staked

**Occurences**: 2 times

1. `RankedBattle::_addResultPoints` function at line 465, RankedBattle.sol.
2. `RankedBattle::_addResultPoints` function at line 500, RankedBattle.sol.

**Impact**:
This could cause wrong information to users or stakeholders if the `globalStakedAmount` doesnt update as it should

**Proof Of Code & Added Mitigation**:

```diff
            /// If the user has stake-at-risk for their fighter, reclaim a portion
            /// Reclaiming stake-at-risk puts the NRN back into their staking pool
            if (curStakeAtRisk > 0) {
                _stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, tokenId, fighterOwner);
                amountStaked[tokenId] += curStakeAtRisk;
                // Added code below as solution
+                 globalStakedAmount += curStakeAtRisk;
            }
```

```diff
{
                /// If the fighter does not have any points for this round, NRNs become at risk of being lost
                bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
                if (success) {
                    _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
                    amountStaked[tokenId] -= curStakeAtRisk;
                     // Added code below as solution
+                    globalStakedAmount -= curStakeAtRisk;
                }
            }
```

globalStakedAmount should be updated to keep proper records of the storage values

## [L-2] `RankedBattle::_addResultPoints` function does not emit events `Staked` or `Unstaked` when `amountStaked[tokenId]` is increased or reduced.

**Occurences**: 2 times

1. `RankedBattle::_addResultPoints`, line 465 RankedBattle.sol
2. `RankedBattle::_addResultPoints`, line 500 RankedBattle.sol

**Impact**: Doesnt follow proper practices and doesnt help readability of code and loss of information

```diff
            if (curStakeAtRisk > 0) {
                _stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, tokenId, fighterOwner);
                amountStaked[tokenId] += curStakeAtRisk;

+               emit Staked(msg.sender, amount)
            }
```

```diff
else {
                /// If the fighter does not have any points for this round, NRNs become at risk of being lost
                bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
                if (success) {
                    _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
                    amountStaked[tokenId] -= curStakeAtRisk;
+                    emit unStaked(msg.sender, amount)
                }
            }
```

**Recommended Mitigation:**

1. A check such as `require(fighterType == 1 || fighterType == 0, "fighterType should be 1 or 0")` can be put to make sure admin gets to put the right input.



## [L-3] Casting a `uint32` value to a uint256 storage at `GameItems::_replenishDailyAllowance` contract is not best practise.

**Occurence**:
- GameItems.sol, Line 314, function `_replenishDailyAllowance`
**Impact**: Wrong practise to cast a uint256 which is a `mapping(address => mapping(uint256 => uint256)) public dailyAllowanceReplenishTime;` to a uint32 timestamp

```javascript
  function _replenishDailyAllowance(uint256 tokenId) private {
        allowanceRemaining[msg.sender][tokenId] = allGameItemAttributes[tokenId].dailyAllowance;

        dailyAllowanceReplenishTime[msg.sender][tokenId] = uint32(block.timestamp + 1 days);
    }    
```

**Recommended Mitigation:**
1. mapping should be changed to `mapping(address => mapping(uint256 => uint32)) public dailyAllowanceReplenishTime;`

## [G-1] `attributes.length` at `AiArenaHelper::addAttributeProbabilities` should be declared as a constant = 6 or reduces to an uint8 in the loops to save gas.

**Occurences**: 3 times

1. `AiArenaHelper::addAttributeProbabilities`, line 138 AiArenaHelper.sol
2. `AiArenaHelper::addAttributeDivisor`, line 72 AiArenaHelper.sol
3. `AiArenaHelper::createPhysicalAttributes`, line 98 AiArenaHelper.sol

**Impact**: Takes more gas

```javascript
   uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
```

**Recommended Mitigation:**

1. attributes.length should be saved as a constant at the beginning of the contract to save gas
2. if you decide not to make attributes.length a constant, it should be initialized as a uint8 since we know the value is very small in the function instead of uint256

## [G-2] Right shift or Left shift instead of dividing or multiplying by powers of two

**Occurences**: 1 time

1.  Neuron.sol:37

**Impact**: Uses more gas

```javascript
    /// @notice Initial supply of NRN tokens to be minted to the treasury.
    uint256 public constant INITIAL_TREASURY_MINT
    10**18 * 10**8 * 2;
```

**Recommended Mitigation:**

1. you can re-write the constant as

```solidity
uint256 public constant INITIAL_TREASURY_MINT
   10**18 * 10**8 << 1 ;
```

## [I-1] Probabilities can be initialized at contract `AiArenaHelper::addAttributeProbabilities` as a 6 length array

**Impact**: None, better practise

```javascript
  function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
    }
```

**Recommended Mitigation:**

1. `uint8[][] memory probablities` array can be initialized as `uint8[6][]` for easy reading and best practices

## [I-2] Line 209 & 210 in FighterFarm::claimFighters does not follow CEI, which is not best practise, both lines should be moved to the bottom of the function after fighter is created

**Impact**: None, not good practise

```diff
    function claimFighters(
        uint8[2] calldata numToMint,
        bytes calldata signature,
        string[] calldata modelHashes,
        string[] calldata modelTypes
    )
        external
    {
        bytes32 msgHash = bytes32(keccak256(abi.encode(
            msg.sender,
            numToMint[0],
            numToMint[1],
            nftsClaimed[msg.sender][0],
            nftsClaimed[msg.sender][1]
        )));
        require(Verification.verify(msgHash, signature, _delegatedAddress));
        uint16 totalToMint = uint16(numToMint[0] + numToMint[1]);
        require(modelHashes.length == totalToMint && modelTypes.length == totalToMint);
-        nftsClaimed[msg.sender][0] += numToMint[0];
-        nftsClaimed[msg.sender][1] += numToMint[1];
        for (uint16 i = 0; i < totalToMint; i++) {
            _createNewFighter(
                msg.sender,
                uint256(keccak256(abi.encode(msg.sender, fighters.length))),
                modelHashes[i],
                modelTypes[i],
                i < numToMint[0] ? 0 : 1,
                0,
                [uint256(100), uint256(100)]
            );
        }
+        nftsClaimed[msg.sender][0] += numToMint[0];
+        nftsClaimed[msg.sender][1] += numToMint[1];
    }
```

## [I-3] Wrong comment on code

**Occurences**: 1 time

1. Line 51, MergingPool.sol -> `fighterPoints` mapping;

**Impact**: false code information.

```javascript
    /// @notice Mapping of address to fighter points.
    mapping(uint256 => uint256) public fighterPoints;
```

**Recommended Mitigation:**

1. comment should be change from `address to fighter points` to `tokenId to fighter points`

## [I-4] `FighterFarm::incrementGeneration` function should check fighterType to be either zero or 1

**Occurences**: 1 time

1. `FighterFarm::incrementGeneration`, line 129 FighterFarm.sol

**Impact**: Doesnt follow proper practices and doesnt help readability of code

```javascript
    function incrementGeneration(uint8 fighterType) external returns (uint8) {
        require(msg.sender == _ownerAddress);
        generation[fighterType] += 1;
        maxRerollsAllowed[fighterType] += 1;
        return generation[fighterType];
    }
```

**Recommended Mitigation:**

1. A check such as `require(fighterType == 1 || fighterType == 0, "fighterType should be 1 or 0")` can be put to make sure admin gets to put the right input.
