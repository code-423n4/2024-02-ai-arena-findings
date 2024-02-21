### RankedBattle ```globalStakedAmount``` variable is not correctly tracked
## Impact
The variable will incorrectly tracked and displayed on the front end and when viewed on the blockchain. It will be perpetually increasing and include an inflated value that will get more and more inaccurate over time.

## Vulnerability Explanation

The variable ```globalStakedAmount``` is only incremented inside ```stakeNRN```
```
    function stakeNRN(uint256 amount, uint256 tokenId) external {
        //...
        if (success) {
            //...
@>          globalStakedAmount += amount;
```
And is only decremented inside ```unstakeNRN```.

```
   function unstakeNRN(uint256 amount, uint256 tokenId) external {
        //...
@>      globalStakedAmount -= amount;
```

The issue is, the ```unstakeNRN``` function doesn't take into account NRN transferred to the ```StakeAtRisk``` contract. When a fighter loses a match and has no points available, a portion of his NRN is at risk of being lost. All at risk NRN that is not reclaimed by the time the round ends will still be included in the ```globalStakedAmount``` value, causing it to be overinflated and not representative of the actual amount staked.

## Proof of concept
```
globalStakedAmount = 0
amountStaked = 0
stakeAtRisk = 0
```
1. player1 stakes 1000 NRN
```
globalStakedAmount = 1000
amountStaked = 1000
stakeAtRisk = 0
```
2. player1 loses 3 times
```
globalStakedAmount = 1000
amountStaked = 997
stakeAtRisk = 3
```
3. player1 unstakes his remaining amount
```
globalStakedAmount = 3
amountStaked = 0
stakeAtRisk = 3
```
4. the round ends
```
globalStakedAmount = 3
amountStaked = 0
stakeAtRisk = 0
```
globalStakedAmount is not correctly tracking the amount of NRN at risk. This means it will be perpetually increasing
## Tools Used
VIM
## Recommended Mitigation Steps
Whenever NRN is placed at risk, subtract the at risk amount from ```globalStakedAmount```.

Whenever at risk NRN is won back, add it back to ```globalStakedAmount```.









