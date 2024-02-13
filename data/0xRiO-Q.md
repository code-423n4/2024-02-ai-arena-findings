# Issue in `mint` Function

## Description
The `mint` function in the contract has a comparison operator issue in the `require` statement, where it checks if the total supply plus the minted amount exceeds the maximum supply.

## Error
The cause of the issue is the incorrect usage of the comparison operator in the `require` statement. The operator `<` is used instead of `<=`, which results in incorrect validation of the minted amount against the maximum supply.
```
    function mint(address to, uint256 amount) public virtual {
        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }
```

## Impact
- Actual supply will always be less than the MAX_SUPPLY of the token which can have impact on economic price of the token.

## Correction
To address this issue, the comparison operator in the `require` statement should be changed from `<` to `<=` to correctly validate if the total supply plus the minted amount is less than or equal to the maximum supply.

# User can never lose stake even if they have very little points

## Instance 
- This check in RankedBattle::addBalance function make it impossible to lose on their stake if they have very little points which let them perform poorly with some player which will let the opponents to have a high elo and to get high share on Pool.
[These opponents can also be their different Id, which will be end up in a win situation for the user]
```
                if (points > accumulatedPointsPerFighter[tokenId][roundId]) {
                    points = accumulatedPointsPerFighter[tokenId][roundId];
                }
```
