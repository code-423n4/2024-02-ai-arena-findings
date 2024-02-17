## [L-1] Add 0 check in `AiArenaHelper::addAttributeDivisor()`
 Recommended to add 0 checks for `defaultAttributeDivisor[i]` in `AiArenaHelper::addAttributeDivisor` since it can lead to division by 0 error and  block users from creating new fighters or rerolling existing ones. The `attributeToDnaDivisor` is used to calculate the rarity rank by dividing the dna.

## [L-2] PUSH0 opcode not supported on Arbitrum
The contracts use solidity version `>=0.8.0 <0.9.0`, but in version `0.8.20` the PUSH0 opcode is introduced. This opcode is not present on Arbitrum.
It is recommended to set the compiler version to
`0.8.19` or earlier, to prevent any mistakes with the evm version in the future.

## [L-3] Set max Bps loss in `RankedBattle::setBpsLostPerLoss`
The max value of basis points by default is 10_000 and is currently set to 0.1 % = 10 bps. This can be changes in `RankedBattle::setBpsLostPerLoss()` only by the admin. It would be really nice to have a check that the new bsp lost per round are less than 10_000. 

## [N-1] Do not pass random bytes in mint function as it can lead to unexpected behavior
The `GameItems::mint` function is used to mint game items. After some checks and operations in calls the default `_mint` function and passes 'random' bytes `_mint(msg.sender, tokenId, quantity, bytes("random"));`
### Recomendation:
Use `_mint(msg.sender, tokenId, quantity, "");` instead

## [N-2] Wrong natspec
In `MergingPool.sol`  The mapping is actually tokenId to fighter points
```
/// @notice Mapping of address to fighter points. 
mapping(uint256 => uint256) public fighterPoints;
```

