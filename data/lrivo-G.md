# [G-1] Consider caching `roundId` in a memory variable inside `RankedBattle::_addResultPoints()` to avoid storage reads

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L416

## Description
`RankedBattle::_addResultPoints()` makes use of this variable multiple times inside his logic so, caching it in a memory variable, could help saving some gas.

# [G-2] Useless storage read when emitting `VoltageManager::VoltageRemaining` event inside `VoltageManager::useVoltageBattery`

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L98

## Description
The value of the `voltage` parameter will always be 100 in this case so it is a waste of gas to read that from storage.

## Recommended Mitigation

Either make the literal 100 a constant variable (like `FULL_VOLTAGE_CAPACITY`) and emit that in the event, otherwise just emit the literal 100 instead of reading it from the `ownerVoltage` mapping.

```js
ownerVoltage[msg.sender] = 100; // this can become a constant variable
emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
```

## [G-3] Avoid reading the supposedly same array length multiple times

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L243

## Recommended Mitigation

Cache the length of the first array and use it to compare with the rest.

```
uint256 len = mintpassIdsToBurn;
require(
    mintPassDnas.length == len && 
    fighterTypes.length == len && 
    modelHashes.length == len && 
    modelTypes.length == len
)
```
