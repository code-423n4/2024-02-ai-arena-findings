## $NRN can't be minted to max capacity
`Neuron::mint` will fail if the mint amount places the supply at the exact maximum capacity.
[https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L156](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L156)
```solidity
require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
```

If `totalSupply() + amount == MAX_SUPPLY` the assertion will fail, even though it shouldn't as `MAX_SUPPLY` is valid. To fix this, I suggest changing the `<` operator to `<=`.

## Every battle will be counted twice

`RankedBattle::updateBattleRecord` is called twice for every battle, once for each user involved in the battle. However, `RankedBattle::totalBattles` is incremented every time `RankedBattle::updateBattleRecord` is called leading to double counting of battles.

[https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L348](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L348)

As a fix, I suggest incrementing `RankedBattle::totalBattles` only when `RankedBattle::updateBattleRecord` is called for the initiator.

## Neuron::burnFrom removes unlimited approval.

In OpenZeppelin's `ERC20` implementation `type(uint256).max` is used to signal that a spender can spend all the tokens an account has and will ever have. `Neuron::burnFrom` ignores this special value and subtracts the burnt amount from `type(uint256).max` thus removing the unlimited approval the spender had.

[https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L201-L203](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L201-L203)

As a fix, I suggest removing `Neuron::burnFrom` and `Neuron::burnFrom` and instead inheriting them from OpenZeppelin's `ERC20Burnable`.

## GameItems::remainingSupply returns 0 for infinite items.

`GameItems` can be a finite or unlimited supply. In cases where an item supply is unlimited, `GameItemAttributes::itemsRemaining` will be set to 0. This makes it so that when `GameItems::remainingSupply` returns 0, it can either mean that there aren't any items left or that there's an infinite supply. This can easily be a source of bugs for front-ends or other users of `GameItems::remainingSupply`.

[https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L279-L281](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L279-L281)

Instead of returning `GameItemAttributes::itemsRemaining` when the supply is unlimited, I suggest returning a special value to indicate there's no supply limit.



