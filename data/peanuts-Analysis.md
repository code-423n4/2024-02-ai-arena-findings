### Summary

- Users can get their fighters, which are ERC721 tokens, through a mint pass or by winning ranked battles
- These fighters have unique traits and abilities which can be rerolled
- These fighters will be used in ranked battles. In every battle, points can be earned or deducted
- These fighters can be trained to be stronger. If they are stronger, they can battle in a higher elo pool, which earns more points
- The ranked battle is a one on one battle. The fighters will be matched against one of similar elo. 
- A user has to stake some tokens in order to earn points. If the user wins a battle, he gets some points. If the user loses the battle, he loses some points
- If the user has no points to lose, then his staked tokens will be at risk. The user can claim back the tokens if he wins a battle.
- There can be many battles in a round. Once the round is over, at risk tokens will be sent to the treasury
- Users can earn NRN tokens from the points that they accumulated. Every battle has a set number of NRN tokens to be distributed proporitionally to the number of points earned

- Every battle uses 10 voltage, which is recharged to 100 everyday. Only the challenger (the person who clicks battle) will use 10 voltage. If the user decides to play more than 10 battles in a day (using more than 100 voltage), he has the option to buy and use a battery, which will recharge the voltage back to 100
- The protocol also has an ERC1155 token shop where the user can buy stuff to help with their battles (one of the stuff is the battery)

### Codebase quality analysis

##### [GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol)

| Contract    | Function                   | Access    | Comments                                                            |
| ----------- | -------------------------- | --------- | ------------------------------------------------------------------- |
| MergingPool | transferOwnership          | onlyOwner | No two step, no zero address check                                  |
| MergingPool | adjustAdminAccess          | onlyOwner | owner can control all isAdmin access                                |
| MergingPool | adjustTransferability      | onlyOwner | items can be transferred at the discretion of the owner             |
| MergingPool | instantiateNeuronContract  | onlyOwner | no zero address check                                               |
| MergingPool | mint                       | public    | all user has a daily allowance on all items                         |
| MergingPool | setAllowedBurningAddresses | onlyAdmin | must be careful with this, burner address can burn anyone's ERC1155 |
| MergingPool | setTokenURI                | public    | does changing URI affect any EIP specs?                             |
| MergingPool | createGameItem             | onlyAdmin | tokenID is set sequentially                                         |
| MergingPool | burn                       | public    |                                                                     |
| MergingPool | getAllowanceRemaining      | public    |                                                                     |

- controls the minting and allowance of every user
- Every ERC1155 item has a finite or infinite supply. Every user has a daily allowance. A user can only buy x number of ERC1155 item a day.

**Checks**

- Checked user unable to mint when price is 0 (game item not set)
- Checked allowance is all computed properly. If token is finite, when bought, the supply is decreased properly.
- Checked daily allowance replenish, and user cannot game the system
- Checked Neuron is actually paid to the Neuron contract

##### [MergingPool.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol)

| Contract    | Function               | Access       | Comments                                                |
| ----------- | ---------------------- | ------------ | ------------------------------------------------------- |
| MergingPool | transferOwnership      | onlyOwner    | No two step, no zero address check                      |
| MergingPool | adjustAdminAccess      | onlyOwner    | owner can control all isAdmin access                    |
| MergingPool | updateWinnersPerPeriod | isAdmin      | no upper bound of winnersPerPeriod, also no lower bound |
| MergingPool | pickWinner             | isAdmin      | Unsure how the winners are chosen. Points will be reset |
| MergingPool | claimRewards           | winner       | Nested loop can be dangerous, may be OOG                |
| MergingPool | getUnclaimedRewards    | view         | Check whether the address has claimable rewards         |
| MergingPool | addPoints              | rankedbattle | add points for fighter and total points                 |
| MergingPool | getFighterPoints       | view         | get accumulated points for fighter                      |

- This contract focuses on getting the winners for each round and giving them an NFT

- All function has proper access controls
- ClaimRewards cannot be gamed as it is a protected function

**Potential Issues**

- If the winner doesn't claim and the rounds continue, he might get an out of gas error.
- updateWinnersPerPeriod has no upper or lower bound
- The roundID is not synced to the Ranked Battle roundID. The points may not work as intended

##### [Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol)

| Contract | Function          | Access                 | Comments                                                                |
| -------- | ----------------- | ---------------------- | ----------------------------------------------------------------------- |
| Neuron   | transferOwnership | onlyOwner              | No two step, no zero address check                                      |
| Neuron   | addMinter         | onlyOwner              | Sets minter                                                             |
| Neuron   | addStaker         | onlyOwner              | Staker is the MergingPoolContract                                       |
| Neuron   | addSpender        | onlyOwner              | Spender is the GameItems contract                                       |
| Neuron   | adjustAdminAccess | onlyOwner              | owner can control all isAdmin access                                    |
| Neuron   | setupAirdrop      | isAdmin                | Admin sets the amount of NRN to be given away from the treasury address |
| Neuron   | claim             | only airdrop recipient | Claims airdrop                                                          |
| Neuron   | mint              | onlyMinter             | Cannot mint more than 1B supply                                         |
| Neuron   | burn              | public                 | Anyone can burn                                                         |
| Neuron   | approveSpender    | onlyOwner              | Give spender approval to use tokens                                     |
| Neuron   | approveStaker     | onlyOwner              | Give staker approval to use tokens                                      |
| Neuron   | burnFrom          | onlyOwner              | burn from an approved address                                           |

- 1 billion total supply of NRN (Neuron)
- 700 million is minted at constructor, 200 million goes to the treasury and 500 million goes to the contributor.
- A classic ERC20 token contract with additional functions. Take note of setupAirdrop because there is no limit, a malicious admin can airdrop the whole treasury amount to a recipient.

**Checks**

- All important functions have the correct access control
- All burn functions cannot be called by anyone except approved address or self.
- Checked that mint amount cannot be more than max supply

**Potential Issues**

- No check for address0, no two step ownership transfer. (Low)
- Treasury approval to recipient can be dangerous. Have to look deeper to find an impact.

##### [RankedBattle.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol)

- `transferOwnership()`, `adjustAdminAccess()`, `setGameServerAddress()`, `setStakeAtRiskAddress()`, `instantiateNeuronContract()`, `instantiateMergingPoolContract()` functions all rely on the ownerAddress and changes the address accordingly

| Contract     | Function                 | Access     | Comments                                                                                               |
| ------------ | ------------------------ | ---------- | ------------------------------------------------------------------------------------------------------ |
| RankedBattle | setRankedNrnDistribution | Admin      | No upper bound of new distributions                                                                    |
| RankedBattle | setBpsLostPerLoss        | Admin      | No upper bound of bps loss                                                                             |
| RankedBattle | setNewRound              | Admin      | setNewRound() can be called at any time if the points is >0                                            |
| RankedBattle | stakeNRN                 | User       | if user stake nrn, the fighter cannot be transferred. staking factor is calculated here                |
| RankedBattle | unstakeNRN               | User       | staking factor is updated here. User can unstake at any time                                           |
| RankedBattle | claimNRN                 | User       | claimNRN can lead to OOG if the roundId is too high                                                    |
| RankedBattle | updateBattleRecord       | GameServer | burns the voltage of challenger, and update records. Only run if there is amountStaked and stakeAtRisk |
| RankedBattle | \_addResultPoints        | internal   | burns the voltage of challenger, and update records                                                    |

- Main part of the whole protocol, users must stake some tokens in order to earn points. `UpdateBattleRecord` is called by GameServer, which is an unknown entity.
- Points are used to distribute NRN every round. 
- Users have the risk of losing NRN tokens if they lose their battles. They are able to claim back the NRN tokens if they win their battles before the round is over.

**Checks**

- Checked all sensitive functions have access control
- Checked all integration with other contracts works well
- Checked fighters cannot be transferred if there are staked tokens
- Checked that the 5 outcomes of the ranked battle is coded properly

```
1) Win + no stake-at-risk = Increase points balance
2) Win + stake-at-risk = Reclaim some of the stake that is at risk
3) Lose + positive point balance = Deduct from the point balance
4) Lose + no points = Move some of their NRN staked to the Stake At Risk contract
5) Tie = no consequence
```

- Checked that the NRN is deposited in to the correct contracts

##### [StakeAtRisk.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol)

| Contract    | Function            | Access              | Comments                                                                   |
| ----------- | ------------------- | ------------------- | -------------------------------------------------------------------------- |
| StakeAtRisk | setNewRound         | RankedBattleAddress | Collects all stakeAtRisk to the treasury address and increases the roundId |
| StakeAtRisk | reclaimNRN          | RankedBattleAddress | From ranked battle, transfer nrn back to the ranked battle contract        |
| StakeAtRisk | updateAtRiskRecords | RankedBattleAddress | updates the `stakeAtRisk` variable of the fighterOwner                     |

- A relatively small contract that interacts with RankedBattle.sol
- Most important variable is the `stakeAtRisk`. If the round is changed, the `stakeAtRisk` tokens will be swept to the treasury address.

**Checks**

- Checked all functions have appropriate access control
- Checked interaction with RankedBattle is correct
- Checked all variables are increased and decreased properly
- Checked transfer of neuron is correct

**Potential Issues**

- NIL

##### [VoltageManager.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol)

| Contract       | Function                     | Access                          | Comments                                                              |
| -------------- | ---------------------------- | ------------------------------- | --------------------------------------------------------------------- |
| VoltageManager | transferOwnership            | onlyOwner                       | No two step, no zero address check                                    |
| VoltageManager | adjustAdminAccess            | onlyOwner                       | Can set own access to false, controls all isAdmin access              |
| VoltageManager | adjustAllowedVoltageSpenders | isAdmin                         | sets any address to spend voltage                                     |
| VoltageManager | useVoltageBattery            | msg.sender                      | msg.sender must have an ERC1155 battery                               |
| VoltageManager | spendVoltage                 | self / allowed voltage spenders | Can call here and spend all voltage, replenish voltage if time passed |
| VoltageManager | \_replenishVoltage           | private                         | Replenish time is 1 day, use of uint32 instead of uint40              |

- A contract that focuses on recharging and usage of batteries. I believe one match uses 10 voltage, and if all the voltage is used, the player can either wait for the replenish time (1 day) or use a battery.
- Battery can be used even at 99 voltage or 0 voltage, to reset the player's voltage to 100.

**Checks**

- All functions work as intended. All access control is correct.
- Battery and replenish works as intended.
- External intergration to use game item and burn is correct, although tokenID must be 0.

**Potential Issues**

- Lack of zero address check (Low)
- Lack of two step ownership transfer (Low)
- ERC1155 battery MUST have tokenID == 0. (The GameItems contract doesn't enforce this) (Low)

### Centralization Risks

- In Neuron.sol, the MINTER_ROLE can mint as much tokens as he wants, up to total supply. This MINTER_ROLE is set by the `_ownerAddress()`.
- In Neuron.sol, the admin can set up any amount of airdrops to any amount of recipient
- In RankedBattle.sol, the owner controls all the contract addresses.
- In RankedBattle.sol, the admin can call `setNewRound()` any time if the totalAccumulatedPoints > 0.
- In RankedBattle.sol, the admin can set the bps lost to > 100%
- In RankedBattle.sol, the admin controls the rankedNrnDistribution
- In GameItems.sol, the admin is incharge of setting the price of the game items in `createGameItem()`
- In GameItems.sol, the admin can burn all the ERC1155 tokens of any users
- In MergingPool.sol, the admin can set the winners of NFTs through `pickWinner()`
- In MergingPool.sol, the admin sets the number of winners per period, and can set to any amount (including zero)

### Systemic Risks

- At some point in time, the MAX_SUPPLY will be reached. All battles will not be incentivzed anymore
- It is unknown how winners are picked in MergingPool and how many winners are picked each time. 
- The rounds can be changed at any time in `RankedBattle.setNewRound()`. There is no fixed schedule

### Architecture Review

##### `Design Patterns and Best Practices:`

- Modifiers should be used if code is repeated.
- Code does not adhere to CEI interaction when transferring tokens, although this does not cause any major issues

##### `Code Readability and Maintainability:`

- Code is pretty manageable and all external calls to different contract is well connected.
- Code does not use any external intergration except OpenZeppellin, which makes it easier to comprehend
- Code is written cleanly. Function names are easy to understand. There is no complex math algorithm or usage of assembly.
- Some parts of the code is not explained well enough (like how eloFactor is calculated, how is a winner chosen in MergingPool)

##### `Error Handling and Input Validation:`

- Events and Error messages are easy to understand. 
- Input is not validated. Important functions lack a lower and upper bound, and functions take takes in an address change lacks a zero address check.

##### `Interoperability and Standards Compliance:`

- Good knowledge of the ERC20 and ERC1155 standard. Creation of ERC1155 tokens is properly done
- ERC721 functions are overridden, but MAX_FIGHTERS_ALLOWED can still be bypassed by calling `safeTransferFrom()` in the original ERC721 contract (with 4 parameters)

##### `Testing and Documentation:`

- Extensive tests (unit, integration tests) done well.
- Documentation (whitepaper, github) is plentiful and includes quality reasoning at every juncture of the protocol. 
- The AI training part and its relation to Solidity requires more explanation (although I think all those are done off chain in other servers)

##### `Upgradeability:`

- Protocol is not intended to be upgradeable, but contracts can be redeployed and rerouted easily through all the onlyOwner functions.

##### `Dependency Management:`

Protocol relies on external libraries like OpenZeppelin. Protocol should keep an eye on vulnerabilities that affects those external integrations, and make changes where necessary.

Overall, great architecture from the protocol, slight changes would be to the written code itself (using modifiers for repeated code, checking zero values, checking overflows/underflows etc).

### Time spent:
20 hours