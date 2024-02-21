## [L-01] globalStakedAmount is greater than actual staked amount

The `globalStakedAmount` state variable in `RankedBattle.sol` keeps track of the overall staked amount in the protocol. It is increased by staking and decreased by unstaking.
However, it does not take into account the stake moved to the `StakeAtRisk` contract, a part of which will be sent to the treasury. 

As such, every round the `globalStakedAmount` becomes a fraction larger than the actual staked amount and this discrepancy will only increase over time. 

This is currently only a Low since the variable is not used in any calculations.

## [L-02] Incomplete require check in redeemMintPass

The require statement in `redeemMintPass` checks that all the arrays have the same length, but does not include `iconsTypes`. 

As a result, it is possible to input an `iconTypes` array with a shorter length, which will case een Out-Of-Bounds error and revert the function. 


``` diff 
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
        require(
            mintpassIdsToBurn.length == mintPassDnas.length && 
            mintPassDnas.length == fighterTypes.length &&
+           fighterTypes.length == iconsTypes.length &&   
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

## [L-03] Incomplete require check in GameItems.mint

The 4th require statement in `GameItems.mint` checks that the `quantity` demanded does not exceed the dailyAllowance or the allowanceRemaining. 
Yet the first part of the require only checks if the user can replenish his allowance, **not** that the `quantity` is equal or lower than the dailyAllowance. 

As a consequence, if `quantity` = 100, `dailyAllowance` = 10 and `dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp` == true, then it will pass the require statement.   

This will cause an underflow and revert on `allowanceRemaining[msg.sender][tokenId] -= quantity;`

``` diff
        require(
-            dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp || 
+            (dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp && quantity <= allGameItemAttributes[tokenId].dailyAllowance ) || 
            quantity <= allowanceRemaining[msg.sender][tokenId]
        );
```


## [L-04] GameItems.remainingSupply will return 0 for a game item with infinite supply

When a game item is created, if `finiteSupply == true` then the `itemsRemaining` is set to a specific number. 
However, when `finiteSupply == false` then the `itemsRemaining` is set to 0. This is confirmed in the `Deployer` script for the battery game item. 

As such, when a player calls `remainingSupply` on a game item with infinite supply, he will get 0 as return. Which is obviously an error. 
I would suggest changing the `remainingSupply` to avoid confusion.

``` solidity
    function remainingSupply(uint256 tokenId) public view returns (uint256) {
        if(allGameItemAttributes[tokenId].finiteSupply){
            return allGameItemAttributes[tokenId].itemsRemaining;
        }else{
            return type(uint256).max;
        }
    }

```

## [NC-01] Replenish Voltage when Allowance is sufficient

The `spendVoltage` function in `VoltageManager` does not check if the current voltage is sufficient for the expenditure before calling `_replenishVoltage`.
This means a player can have a maximum voltage of 100 and still be forced to replenish to a 100 when he spend 10 to do one battle. 

The replenish timer is then set every day exactly 24 hours after the first battle, which forces players to keep track of this timer on a daily basis.  

## [NC-02] Arbitrum Decentralisation and Front-running

If it were possible to frontrun the results of a battle then this would have severe negative consequences for the protocol. 
Current Arbitrum is centralised and frontrunning is impossible. 

However, decentralision is planned by Arbitrum and will become reality. At which point the protocol will have a serious problem. 





