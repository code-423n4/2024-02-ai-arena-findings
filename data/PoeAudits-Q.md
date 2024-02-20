# Low Issues:

## Approve Function Race-Condition in Neuron.sol
Issue Type: Error
Link: https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L171-177
      https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L184-190
      https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L196-204

#### Impact
Users with a non-zero approval balance can frontrun approval transactions to exceed their intended allowance. 

#### Proof of Concept
This is a well documented vulnerability regarding race-conditions. 
https://github.com/crytic/not-so-smart-contracts/tree/master/race_condition

The approve function is vulnerable to frontrunning attacks where the target of the approval first spends their approval balance, before spending their newly approved balance. Quoting Crytic's "not-so-smart-contracts": 

"The ERC20 standard's approve and transferFrom functions are vulnerable to a race condition. Suppose Alice has approved Bob to spend 100 tokens on her behalf. She then decides to only approve him for 50 tokens and sends a second approve transaction. However, Bob sees that he's about to be downgraded and quickly submits a transferFrom for the original 100 tokens he was approved for. If this transaction gets mined before Alice's second approve, Bob will be able to spend 150 of Alice's tokens."

The increaseAllowance and decreaseAllowance in the Openzeppelin implementation explicity mention this:
     ```
     * @dev Atomically increases the allowance granted to `spender` by the caller.
     *
     * This is an alternative to {approve} that can be used as a mitigation for
     * problems described in {IERC20-approve}.
     ```
     
#### Tools Used
Manual Review

#### Recommended Mitigation Steps
Use increaseAllowance/decreaseAllowance instead of approve. 


## Minting Enables Public Use of _replenishDailyAllowance
Issue Type: Input Validation

Link: https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L165-L172

#### Impact
Public use of private function _replenishDailyAllowance for no Neuron cost. Could have minor interactions with future items. 

#### Proof of Concept
The mint function has no check for minting zero items, which normally does not serve a purpose. Viable inputs with no real purpose or usecase are generally avoided. The mint function calls the private function _replenishDailyAllowance to replenish the user's allowance if enough time has passed. By minting zero game items, it costs the user zero Neuron, and updates the state variables allowanceRemaining and dailyAllowanceReplenishTime. This effectively makes the _replenishDailyAllowance function public instead of private while costing the user extra gas used by the mint logic. A PoC is included which adds extra logic to the test/GameItems.t.sol function testMintGameItem which demonstrates how the user can invoke the private function at no Neuron cost. 

```
    function testMintGameItem() public {
        _fundUserWith4kNeuronByTreasury(_ownerAddress);
        _gameItemsContract.mint(0, 2); //paying 2 $NRN for 2 batteries
        assertEq(_gameItemsContract.balanceOf(_ownerAddress, 0), 2);


        ///////////////////////////////////////////////
        /////////////// Added Code ////////////////////

        // Get the daily allowance of the batteries
        (,,,,,uint256 batteryAllowance) = _gameItemsContract.allGameItemAttributes(0);
        console.log("Total Battery Allowance: ", batteryAllowance);
        // Check the allowanceRemaining after purchase
        uint256 remaining = _gameItemsContract.allowanceRemaining(_ownerAddress, 0);
        // Assert the allowance is as expected
        assertEq(remaining, batteryAllowance - 2);
        console.log("Remaining Allowance After Mint: ", remaining);
        
        // Skip to next day
        vm.warp(1 days + 1); 
        // Check allowance remains the same
        uint256 remainingNextDay = _gameItemsContract.allowanceRemaining(_ownerAddress, 0);
        console.log("Remaining Allowance Next Day: ", remainingNextDay);
        uint256 ownerBalanceBefore = _neuronContract.balanceOf(_ownerAddress);
        // Zero mint to set allowanceRemaining and dailyAllowanceReplenishTime variables
        _gameItemsContract.mint(0,0);
        uint256 ownerBalanceAfter = _neuronContract.balanceOf(_ownerAddress);
        // Check allowance has been updated
        uint256 finalRemaining = _gameItemsContract.allowanceRemaining(_ownerAddress, 0);
        console.log("Remaining Allowance After Zero Mint: ", finalRemaining);

        console.log("Owner Balance Before Zero Mint: ", ownerBalanceBefore);
        console.log("Owner Balance After Zero Mint: ", ownerBalanceAfter);
    }

```

```
Logs:
[PASS] testMintGameItem() (gas: 204574)
Logs:
  Total Battery Allowance:  10
  Remaining Allowance After Mint:  8
  Remaining Allowance Next Day:  8
  Remaining Allowance After Zero Mint:  10
  Owner Balance Before Zero Mint:  3998000000000000000000
  Owner Balance After Zero Mint:  3998000000000000000000
```

## Tools Used
Manual Review

## Recommended Mitigation Steps
Add an input validiation check in the mint function for quantity. 
```
require(quantity != 0);
```


## Additional Undocumented Scenario for _addResultPoints
Issue Type: Error

Link: https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L405-L410
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L472-L499

#### Impact
Abusive player strategy to reduce/eliminate stake loss to protocol.

#### Proof of Concept
According to the natspec documentation for _addResultPoints:

```
    /// @dev There are 5 scenarios to be aware of:
    /// @dev 1) Win + no stake-at-risk = Increase points balance
    /// @dev 2) Win + stake-at-risk = Reclaim some of the stake that is at risk
    /// @dev 3) Lose + positive point balance = Deduct from the point balance
    /// @dev 4) Lose + no points = Move some of their NRN staked to the Stake At Risk contract
    /// @dev 5) Tie = no consequence
```
These 5 scenarios do not explicitly take into account the result when the player loses with no points and $amountStaked[tokenId] = 0$. If this is the case, there is effectively no punishment for losing:

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L475-L478
```
    /// Do not allow users to lose more NRNs than they have in their staking pool
    if (curStakeAtRisk > amountStaked[tokenId]) {
        curStakeAtRisk = amountStaked[tokenId];
    }
```

This will transfer 0 Neuron to the _stakeAtRiskAddress on line 493. A user can exploit this by repeatedly playing the game until they accumulate a significant points buffer before staking their Neuron tokens. This makes it infeasible that they will ever lose their stake since they can stop playing when they lose their last points. This then allows the user to stake very large amounts of tokens at effectively no risk due to their points buffer.  


#### Tools Used
Manual Review

## Recommended Mitigation Steps
Consider the additional scenario where users lose while having zero points and no stake-at-risk. One method would be to track the points as an integer instead of a unsigned integer, so points can become more negative only when users with $points <= 0$ and no stake-at-risk continue to lose. 




## QA Issues:

#### Natspec for fighterPoints does not Match Implementation

Issue Type: Error
Link: https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L50-L51

#### Impact
Function definition contradicts comments

#### Proof of Concept

The mapping is a uint256 to uint256 and not an address to a uint256. The mapping is from the tokenId to fighter points
```
    /// @notice Mapping of address to fighter points.
    mapping(uint256 => uint256) public fighterPoints;
```

#### Tools Used
Manual Review

#### Recommended Mitigation Steps
Update the comment to "Mapping of tokenId to fighter points."


