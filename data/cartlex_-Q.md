| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01-mint-function-is-useless-if-dailyallowance-set-to-0) | `mint` function is useless if `dailyAllowance` set to 0. | Low |
| [L-02](#l-02-direct-call-to-spendvoltage-is-useless) | Direct call to `spendVoltage` is useless. | Low |
| [NC-01](#nc-01-lack-of-fightertype-check-in-incrementgeneration-function) | Lack of `fighterType` check in `incrementGeneration` function. | Non-critical |
| [NC-02](#nc-02-lack-of-zero-divisor-check) | Lack of zero divisor check. | Non-critical |
| [NC-03](#nc-03-lack-of-iconstypes-array-length-check) | Lack of `iconsTypes` array length check. | Non-critical |
| [NC-04](#nc-04-unsafe-casting-to-uint16) | Unsafe casting to uint16. | Non-critical |
| [NC-05](#nc-05-no-boundary-for-setting-bpslostperloss-value) | No boundary for setting `bpsLostPerLoss` value. | Non-critical |
| [NC-06](#nc-06-comment-is-misleading) | Comment is misleading. | Non-critical |
| [NC-07](#nc-07-dna-will-always-consist-of-mergingpool-address-when-mintfrommergingpool-function-is-used) | Dna will always consist of `MergingPool` address when `mintFromMergingPool` function is used. | Non-critical |

# [L-01] `mint` function is useless if `dailyAllowance` set to 0.

## Description
In `GameItems` contract when game item is creating it's possible to set `dailyAllowance` value to 0.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L215
 
Due to this `mint` function will always revert if `quantity != 0`. 

Moreover, `mint` function do nothing with `quantity == 0`

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L147

## Recommendation
1. Consider to add require statement in `createGameItem` function.

```diff
    function createGameItem(
        string memory name_,
        string memory tokenURI,
        bool finiteSupply,
        bool transferable,
        uint256 itemsRemaining,
        uint256 itemPrice,
        uint16 dailyAllowance
    ) 
        public 
    {
++      require(dailyAllowance != 0, "InvalidAllowance");
        require(isAdmin[msg.sender]);
        allGameItemAttributes.push(
            GameItemAttributes(
                name_,
                finiteSupply,
                transferable,
                itemsRemaining,
                itemPrice,
                dailyAllowance
            )
        );
        if (!transferable) {
          emit Locked(_itemCount);
        }
        setTokenURI(_itemCount, tokenURI);
        _itemCount += 1;
    }
```

2. Add require that `quantity != 0` in `mint` function.

```diff
function mint(uint256 tokenId, uint256 quantity) external {
        require(tokenId < _itemCount);
++      require(quantity != 0, "Invalid amount");
    ...
}
```


# [L-02] Direct call to `spendVoltage` is useless.

## Description
In `VoltageManager` user can directly call `spendVoltage` function. However it does nothing and only spend user's voltage.

```solidity
function spendVoltage(address spender, uint8 voltageSpent) public {
        require(spender == msg.sender || allowedVoltageSpenders[msg.sender]); // @audit user can call it directly
        if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
            _replenishVoltage(spender);
        }
        ownerVoltage[spender] -= voltageSpent; 
        emit VoltageRemaining(spender, ownerVoltage[spender]);
    }
```

## Recommendation
Consider to restrict users from calling this function and allow only `RankedBattle` contract to call it.

```diff
function spendVoltage(address spender, uint8 voltageSpent) public {
--      require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
++      require(spender == address(_rankedBattleAddress), "Only ranked battle can call");
        if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
            _replenishVoltage(spender);
        }
        ownerVoltage[spender] -= voltageSpent; 
        emit VoltageRemaining(spender, ownerVoltage[spender]);
    }
```


# [NC-01] Lack of `fighterType` check in `incrementGeneration` function.

## Description
There are only two types of fighter in the game. 
However, there is no check that `fighterType` is equal `0` or `1` in `incrementGeneration` function.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L129-L134

## Recommendation
Consider to add check that `fighterType` is equal 0 or 1.
```diff
function incrementGeneration(uint8 fighterType) external returns (uint8) {
        require(msg.sender == _ownerAddress);
++      require(fighterType == 1 || fighterType == 0, "Invalid fighterType");
        generation[fighterType] += 1;
        maxRerollsAllowed[fighterType] += 1;
        return generation[fighterType];
    }
```


# [NC-02] Lack of zero divisor check.

## Description
There is no check than input attribute divisor in not equal to zero.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L73-L74

It could lead to zero division error in `createPhysicalAttributes` function, when `rarityRank` is calculating.

## Recommendation
Consider to additional check that attribute divisor is not equal to zero.

```diff
function addAttributeDivisor(uint8[] memory attributeDivisors) external {
        require(msg.sender == _ownerAddress);
        require(attributeDivisors.length == attributes.length);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
++          require(attributeDivisors[i] != 0, "InvalidDivisor");
            attributeToDnaDivisor[attributes[i]] = attributeDivisors[i];
        }
    }   
```


# [NC-03] Lack of `iconsTypes` array length check.

## Description
In `FighterFarm` contract inside the comments to `redeemMintPass` function says:

```
Each input array must correspond to the same index, i.e., the first element in each 
```

However, there is no check for `iconsTypes` array length.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L233

## Recommendation
Consider to add additional check for iconsTypes array length.

```diff
function redeemMintPass(
        uint256[] calldata mintpassIdsToBurn,
        uint8[] calldata fighterTypes,
        uint8[] calldata iconsTypes, // @audit no check for this array and no natspec for it
        string[] calldata mintPassDnas,
        string[] calldata modelHashes,
        string[] calldata modelTypes
    ) 
        external 
    {
        require(
            mintpassIdsToBurn.length == mintPassDnas.length && 
            mintPassDnas.length == fighterTypes.length && 
            fighterTypes.length == modelHashes.length &&
            modelHashes.length == modelTypes.length &&
++          modelTypes.length == iconsTypes.length
        );
```


# [NC-04] Unsafe casting to uint16.

## Description
In `claimFighters` function there is an unsafe calculation of `totalToMint` variable. 

```solidity
uint16 totalToMint = uint16(numToMint[0] + numToMint[1]);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L207C9-L207C66

Function can revert if the sum `numToMint` array elements is greater than 255.

## Recommendation
Consider to implement the next changes:

```diff
--  uint16 totalToMint = uint16(numToMint[0] + numToMint[1]); 
++  uint16 totalToMint = (uint16(numToMint[0]) + uint16(numToMint[1]));
```


# [NC-05] No boundary for setting `bpsLostPerLoss` value.

## Description
In `RankedBattle` an admin can set `bpsLostPerLoss` using `setBpsLostPerLoss` function.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L226-L229


However, there is no any boundary for the `bpsLostPerLoss` value, therefore it can be any value and `curStakeAtRisk` can be more than user expects.

```solidity
    /// Potential amount of NRNs to put at risk or retrieve from the stake-at-risk contract
    curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L439

## Recommendation
Consider to implement the next changes:

```diff
function setBpsLostPerLoss(uint256 bpsLostPerLoss_) external {
    require(isAdmin[msg.sender]);
++  require(bpsLostPerLoss_ <= 10_000, "InvalidBpsValue");
    bpsLostPerLoss = bpsLostPerLoss_;
}
```


# [NC-06] Comment is misleading.

## Description
In the constructor of `Neuron` contract there is a comment:

```
/// @notice Grants roles to the ranked battle contract. 
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L62


However, it doesn't grant the role to the `RankedBattle` contract during the contruction. 
As can see from the `RankedBattle.t.sol`, roles granted via `addStaker` and `addMinter` functions.

```solidity
_neuronContract.addStaker(address(_rankedBattleContract));
_neuronContract.addMinter(address(_rankedBattleContract));
```

## Recommendation
Consider to remove this commentary.

```diff
--  /// @notice Grants roles to the ranked battle contract. 
```


# [NC-07] Dna will always consist of `MergingPool` address when `mintFromMergingPool` function is used.

## Description
It's possible to create a fighter via `mintFromMergingPool` function. This function internally calls `_createNewFighter` function and pass `dna` as a parameter.

To calculate the `dna` it uses the next formula:

```solidity
uint256(keccak256(abi.encode(msg.sender, fighters.length))),
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L324

As can seen it used `msg.sender` i.e. address of `MergingPool` contract.
However, other functions that call `_createNewFighter` use `user` as `msg.sender`.

## Recommendation
Consider to implement the next changes.

```diff
--  uint256(keccak256(abi.encode(msg.sender, fighters.length))),
++  uint256(keccak256(abi.encode(to, fighters.length))),
```