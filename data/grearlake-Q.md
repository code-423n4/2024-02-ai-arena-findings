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


# 2, Current protocol implement is not compatitive with ability to using new items

## Lines of code
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L208-#L235

## Vulnerability details
As discussed with sponor, it is confirmed that future items will have purpose like battery:

![img](https://i.imgur.com/GsrsJd5.png)

But with current protocol design, it is impossible to create another item that affect on anything in the contract. Moreover, protocol does not implement proxy, so upgrading contract is impossible, so creating new item that can interact with protocol like battery is impossible

## Mitigration steps
Mitigrating this issue require changing so many things, including implement upgradable contracts to make new items to be able to interact with contract like battery