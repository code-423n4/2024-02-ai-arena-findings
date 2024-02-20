## G-1: Redundant calculations

#### Description:
In function [_addResultPoints](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L416) [initial value](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L427) of `points` is 0, and it [only changes](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L444) when `stakeAtRisk` is 0. However, [there is a bunch of calculations](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L449-L468) with `points` that are processed even when `points` are not changed, and those calculations have no reason as the result is 0.

#### Recommendation:
Move the calculations with points inside the `if` statement when the points is changed to non-zero value.

## G-2: Redundant condition

#### Description:
The operator `or` is used in the [require](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L151-L157) statement, the first operand of `or` is `allGameItemAttributes[tokenId].finiteSupply == false`, and the second is `allGameItemAttributes[tokenId].finiteSupply == true && quantity <= allGameItemAttributes[tokenId].itemsRemaining`. The part `allGameItemAttributes[tokenId].finiteSupply == true` is redundant as this is always true because of short-circuit boolean.

## G-3: 'attributeProbabilities' are updated twice

#### Description:
[Constructor](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L41-L52) of `AiArenaHelper` updates `attributeProbabilities` for generation 0 twice - once in function `addAttributeProbabilities` and another time after the function is called.