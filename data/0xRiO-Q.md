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
