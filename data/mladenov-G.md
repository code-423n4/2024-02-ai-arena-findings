# [G-01] The redeemMintPass function can be executed with more than 10 mint pass IDs

### Description

User is able to call `FighterFarm.sol::redeemMintPass` function with more than 10 ids in `mintpassIdsToBurn` array. The function will revert since there's a check in `FighterFarm.sol::_createNewFighter` for how many NFTs user can mint, but user will pay high gas fees.

According to the EVM documentation no matter regardless of whether a transaction succeeds or fails user will pay gas fees.

`The gas fee is the amount of gas used to do some operation, multiplied by the cost per unit gas. The fee is paid regardless of whether a transaction succeeds or fails.`

### Recommendtaion
Add a require statement to check how many mint passes the user wants to redeem. This way, if the user tries to redeem more than 10 mint passes, the function will revert, and the user will pay less gas.

```javascript
function redeemMintPass(
        uint256[] calldata mintpassIdsToBurn,
        uint8[] calldata fighterTypes,
        uint8[] calldata iconsTypes,
        string[] calldata mintPassDnas,
        string[] calldata modelHashes,
        string[] calldata modelTypes
    ) 
        external 
    {
        
       require(mintpassIdsToBurn.length <= 10, 'Cannot redeem more than 10 mint passes'); // Add this check
        require(
            mintpassIdsToBurn.length == mintPassDnas.length && 
            mintPassDnas.length == fighterTypes.length && 
            fighterTypes.length == modelHashes.length &&
            modelHashes.length == modelTypes.length
        );
        for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
            require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));
            _mintpassInstance.burn(mintpassIdsToBurn[i]);
           
            _createNewFighter(
                msg.sender, 
                uint256(keccak256(abi.encode(mintPassDnas[i]))), 
                modelHashes[i], 
                modelTypes[i],
                fighterTypes[i],
                iconsTypes[i],
                [uint256(100), uint256(100)]
            );
        }
    }
```


### Tools used
- Manual Review
- Foundry

# [G-02] `setupAirdrop` function stores the length of recipients in a separate variable

### Description
Since we **don't** use `recipients.length` multiple times in the `setupAirdrop` function, it's cheaper to use `recipients.length` directly in the for loop without creating a separate variable. This way, the execution of the function will be cheaper.
### Recommendtaion
Use `recipients.length` directly in for loop
```javascript
function setupAirdrop(address[] calldata recipients, uint256[] calldata amounts) external {
        require(isAdmin[msg.sender]);
        require(recipients.length == amounts.length);
        // uint256 recipientsLength = recipients.length; // Remove this line of code
        for (uint32 i = 0; i < recipients.length; i++) {
            _approve(treasuryAddress, recipients[i], amounts[i]);
        }
    }
```


### Tools used
- Manual Review