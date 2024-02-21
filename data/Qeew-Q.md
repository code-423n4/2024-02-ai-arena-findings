1. According to Elo rank (as seen in the documentation)- https://docs.aiarena.io/gaming-competition/ranking-system

If Player A wins, the result is recorded as 1.
If the game ends in a tie, the result is recorded as 0.5.
If Player A loses, the result is recorded as 0.

However, this is not correctly implemented in the code, there is no provision for when the competition ends in tie and as such making it not resulting into a loss or gain of points for the two competing players


Take a look at the three cases (as seen in the doc)

Case 1: Player A Wins

Result: 1
Calculation:
New Elo for Player A: ≈ 1500 + 20 * (1 - 0.360) = 1512.8
New Elo for Player B: ≈ 1600 + 20 * (1 - 1 - 0.640) = 1587.2


Case 2: Player A and Player B Tie

Result: 0.5
Calculation:
New Elo for Player A: ≈ 1500 + 20 * (0.5 - 0.360) = 1502.8
New Elo for Player B: ≈ 1600 + 20 * (1 - 0.5 - 0.640) = 1597.2


Case 3: Player A Loses

Result: 0
Calculation:
New Elo for Player A: ≈ 1500 + 20 * (0 - 0.360) = 1492.8
New Elo for Player B: ≈ 1600 + 20 * (1 - 0 - 0.640) = 1607.2


In the example above, it shows during a tie, the one with high elo rank will loss some points and one with lower elo rank will gain some points

This is not correctly reflected in the code here, no calculation of points for tie result

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L416


2. Inheriting from OpenZeppelin Ownable contract means you are using a single-step ownership transfer pattern. If an admin provides an incorrect address for the new owner this will result in none of the onlyOwner marked methods being callable again.The better way to do this is to use a two-step ownership transfer approach, where the new owner should first claim its new rights before they are transferred.

Recommendation

Use OpenZeppelin's Ownable2Step instead of Ownable


3. A number of functions in the codebase do not revert if zero is passed in
for a parameter that should not be set to zero. 

Example;

- Fighterfarm.sol 

setMergingPoolAddress()


- GameItems.sol

adjustAdminAccess()

Recommendation

Ensure to implement check for Zero parameters(values and addresses)
