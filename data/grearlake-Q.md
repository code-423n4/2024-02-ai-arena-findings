# 1, Fake creating fighter emitting can be make by attacker

## Links to affected code
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L53-#L62

## Vulnerability details
Function `FighterOps.fighterCreatedEmitter()` is used to emit when new fighter is created:

    function fighterCreatedEmitter(
        uint256 id,
        uint256 weight,
        uint256 element,
        uint8 generation
    ) 
        public 
    {
        emit FighterCreated(id, weight, element, generation);
    }

But it is set to public, so anyone call this can make other everyone confusing when tracking for emitting

## Mitigration steps:
Since it only be called in `FigherFarm._createNewFighter()` function, this function should be restricted to be only called by `FigherFarm` contract.