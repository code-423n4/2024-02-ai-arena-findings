## User can sabotage `_addResultPoints`

### Description
User can make a bot which executes `spendVoltage()` on every block.timestamp + 1 days with `voltageSpent = 100` which would make the `_addResultPoints` revert every time when the user is the initiator. 

On every battle, the `updateBattleRecord` will be executed for both sides - winner and losser. However, a malisious user can sabotage the update of the points on his side nevermind win or loss. That would lead to incorrect value of the `totalAccumulatedPoints` which are used in the calculations for how much NRN tokens user can claim inside `claimNRN`. 

### Recommendation 
Only `allowedVoltageSpenders` should be allowed to execute `spendVoltage()`


## User can execute `reRoll` with different fighterType

### Description
In `reRoll` there is an argument for the fighterType, but there is no validation if the passed value is the correct fighter type. 

A user can reroll his Champion fighter and pass 1 as fighterType (1 is for Dendroid) when rerolling which would lead to manupulation of the dna. If fighterType=1, the `newDna` would be 1, however it should be a hash of the msg.sender, tokenId and numRerolls[tokenId].

```js
        uint256 newDna = fighterType == 0 ? dna : uint256(fighterType);
```

### Recommendation 
Validate the passed `fighterType` 
