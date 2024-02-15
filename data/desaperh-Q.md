# L1: Some victory/combinations can stuck the game until next voltage replenish #

# Lines of code

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L235


# Vulnerability details

## Impact
At the expected end of the round, if `totalAccumulatedPoints` is equal to 0, it will be impossible to start a new round. The game will be stuck until some player can play again with voltage replenish.

## Proof of Concept

The easiest poc is when only two players only tie during a round.
The following test can show this case:
https://github.com/desaperh/AI_ARENA/blob/main/RankedBattle_dowsers.t.sol

More complex cases can be added. The primary challenge lies in modeling Elo modifications after a match.

## Tools Used
Foundry Invariant Testing

## Recommended Mitigation Steps
Without modifying the code, administrators must ensure there is a fighter owned by administrators ready to participate in a match at the end of the round. Conducting some matches will effectively resolve the situation.
Alternative way : 
https://github.com/desaperh/AI_ARENA/blob/main/RankedBattle_modified.sol
The value 1 can be set to a more suitable value for rewards calculation


## Assessed type

Other