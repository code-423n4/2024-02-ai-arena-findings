#  **Advanced Analysis Report for <img width= 150px src="https://gist.github.com/assets/112232336/d060bb61-2839-4d5c-a0b3-382d8ec0079d" alt="Logo">**
[Audit approach](#1-audit-approach)

[Brief Overview](#2-brief-overview)

[Roles](#3-roles)

[Scope and Architecture Overview](#4-scope-and-architecture-overview)

[Codebase Overview](#5-codebase-overview)

[Centralization Risks](#6-centralization-risks)

[Systemic Risks](#7-systemic-risks)

[Recommendations](#8-recommendations)

[Conclusions](#9-conclusions)

[Resources](#10-resources)

***

## **1. Audit approach**

- **Documentation Dive**: A comprehensive analysis of provided docs was conducted to understand protocol functionality, key points were noted, ambiguities were discussed with the devs, and possible risk areas were mapped.

- **Static Analysis & Linter Sweep**: Initial vulnerability and error detection was performed using static analyzers and linters and known issues and false positives were excluded.

- **Code Inspection**: Manual review of each contract within defined sections was conducted, testing function behavior against expectations, and working out potential attack vectors. Vulnerabilities related to dependencies and  inheritances, were assessed. Comparisons with similar protocols (including older commits) was also performed to identify recurring issues and evaluate fix effectiveness.

- **Report Compilation**: Identified issues were generated into a comprehensive audit report.

***
## **2. Brief Overview**

- AI Arena is a fighting game where players train and battle AI fighters. AI Arena lets users collect and train AI-powered fighters as NFTs, hone their skills, then send them into battle against other players' champions in a platform fighting game. These fighters are represented as NFTs with unique traits like appearance, abilities, and type. 
- Players can enter their fighters into ranked battles to win rewards in the game's own token called $NRN and points, but stand a risk of losing part of their stake upon losing. To keep the game fair, a StakingFactor, based on the square root of the staked amount to is used for points calculation, each wallet has voltage (energy) replenished daily to 100 and needs to be refilled either by waiting for natural refill time or buying a battery from the store.

***
## **3. Roles**

- Owner - The contract deployer and initial admin of the contracts. He's responsible for setting most of the important protocol parameters and addresses.   

- Admins - The admins can be or be set by the contract owner. His responsibilities include starting new game rounds, setting token uri, pick raffle winners, setting up airdrop recipients and so on.

- Minters - These are addresses responsible for minting the NRN tokens. Ideally, this role is granted to the `RankedBattle` contract, but can also be granted to other addresses as well.

- Burners - These are addresses granted the permission to burn game items or mint passes. The `VoltageManager` and the `FighterFarm` contracts are granted this role to burn game items and mint passes respectively.

- Delegate - The delegated address is the address in charge of signing in messages for claiming Fighter NFTs.

- Stakers - These adresses have the permissions to stake Fighter NFTs and NRN on behalf of the user. This role ideally is granted to the `RankedBattle` contract. 

- Spenders - These are the roles approved to spend user's NRNs and voltage on their behalf. The `FighterFarm` and `GameItems` contract are granted the spender role for the NRN tokens, while the `RankedBattle` contract is allowed to spend voltage on behalf of the user.

***
## **4. Scope and Architecture Overview**

<p align="center">
    Contract Architecture
</p>

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/bfe6f0fa-3157-46f1-b7ba-4925972ed36a"
alt="Contract Architecture">
</p>

***

<h3 align="center">
  <b>Core Game Mechanics</b>
</h3>


#### **[RankedBattle.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol)**

- The `RankedBattle` contract is probably the most important contract in the protcol, handling the process of staking NRN tokens on fighters, tracking battle records, calculating and distributing rewards based on battle outcomes and staked amounts, and allowing claiming of accumulated rewards. Due to its important nature, it also holds a number of important roles in the protocol, which are all granted by the owner including minter and staker roles in the `Neuron` contract, burner role in the `GameItem` contract and so on.

**Round Setup** -  The admin, to start a new round calls the `setNewRound` function which clears the stakes at risk in the previous round and sets the new neruon tokens to distribute to winners. 

**Token staking and unstaking** - Users can stake NRN tokens on their fighter every round by calling the `stakeNRN`, theres also the option to keep staking more, icreasing their stake, hence, their point output upon winning battles. They can also call the `unstakeNRN` function to remove a portion or all of their stake if they so wish. The functions, in theory have been setup to prevent fighter transfer while staking. 

**Battle record update** - Since a large portion of the game mechanics is stored offchain, it becomes important to keep the figher's records up to date so as to properly calculate the user's ranking on the leaderboard and how much rewards/punishment they're entitled to. The `gameServerAddress` is takes care of this, by calling the `updateBattleRecord` function which updates the fighter's wins, losses and ties, updates the points a user is entitled to after each battle.

As a basic breakdown, users gain points for winning, lose points for losing. Upon which after losing all their accumulated point, they begin to lose a portion of their stake (the stakeAtRisk). By winning again, users can reclaim a portion of the stakeAtRisk. No changes are made on ties.If users wish, they can divert a portion of their points into a raffle in the `MergingPool` contract. 

**NRN reward claim** - After a round is ended, winners can call the `claimNRN` function which calculates the amount they're entitled to as a function of their accumulated points. The function mints the NRN tokens to the deserving winners.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/f3fbe3ff-b55c-43b4-b588-3d6ee7236c21" alt="RankedBattle.sol">
</p>

<p align="center">
   sLOC - 300
</p>


#### **[StakeAtRisk.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol)**	

- The `StakeAtRisk` contract is an extension of the `RankedBattle` contract that handles the accounting for the user stakes at risk. Virtually all calls to this contract are expected to originate from the `RankedBattle` contract. 

**Stake sweeping** - Upon a call from the `RankedBattle`'s `setNewRound` function, a call to the `setNewRound` function which sweeps the stakes at risk for the previous round before updating the current round.

**StakeAtRisk reclaim and record update** - When users who have stake at risk finally win games, their updated record in the `RankedBattle` contract makes a call to the `reclaimNRN` function to return the tokens back to the user. It also calls the `updateAtRiskRecords` function which makes an update to the stake at risk records.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/71c752b3-2b68-44aa-b5aa-d210375a06eb" alt="StakeAtRisk.sol">
</p>

<p align="center">
   sLOC - 63
</p>



#### **[MergingPool.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol)**

- The MergingPool contract is the raffle contract through which lucky users can be selected by the contract admin. Selected users can then claim their free fighter NFT.

**Winner selection** - The admin can select certain addresses as the winners of the raffle for each round. They do this by calling the `pickWinner` function.
**Rewards claiming** - The winners can call the `claimRewards` to claim their rewards for multiple rounds, once for each round. THe function mints their NFT to them from the `FighterFarm`

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/0adafe53-f4ea-4dc6-a376-7eceeb32cef3" alt="AiArenaHelper.sol"> 
</p>

<p align="center">
   sLOC - 110
</p>

***

<h3 align="center">
  <b>Fighter Management</b>
</h3>

#### **[FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol)**

- The contract manages the creation and mint new fighters either by claiming, minting from a merging pool or redeeming of mint passes. Users also have the option to reroll if they don't like their fighter attributes. The fighters can also be transferred to other users if its not being staked on.

**Fighter minting** - Fighters can be minted in three major ways through signature, mintpass redemption and winning raffles from the MergingPool. Using a provided signature, users can claim a predetermined number of fighters by calling the `claimFighters` function, passing it a given signature, which allows the user mint the fighters. Users who have a mintpass can burn the mintpass to redeem a fighter by calling the `redeemMintPass` function. Raffle winners who decide to claim their prize in the `MergingPool` contract get their claims transfered to this contract through the `mintFromMergingPool` function. Users not pleased with their minted fighter have the option of calling the `reRoll` function to change the fighter's attributes. 

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/bbf3ea4c-7ce8-4551-82be-47e922cce456" alt="FighterFarm.sol	"> 
</p>

<p align="center">
   sLOC - 327
</p>

#### **[FighterOps.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol)**

- This library is for managing fighters in the AI Arena game.It holds various information about a fighter like its weight, element, id, modelHash, type generation and so on. The `getFighterAttributes`  and the `viewFighterInfo` functions can be called to view these information.
<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/63c81af3-ddc3-4b1c-9321-b9d44ed15ffe" alt="FighterOps.sol"> 
</p>

<p align="center">
   sLOC - 74
</p>

#### **[AiArenaHelper.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol)** 

- This contracts holds and manages a fighter's attributes. Here, contract owner can create physical attributes for a user based on their (fighter) DNA, add and delete attributes and its probabilities and convert the DNA and rarity rank into an attribute probability index.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/bacb9637-3148-4c34-9fec-940cd97147de" alt="AiArenaHelper.sol">
</p>

<p align="center">
   sLOC - 95
</p>

*** 


<h3 align="center">
  <b>Game Assets & Economy</b>
</h3>


#### **[GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol)**

- The GameItems contract holds the support items used while gaming in the AI Arena. Admin can create an item and set its various characteristics, including price and supply. Users can then purchase these items to use in their games while paying a fee. Currently, no game item has been created, but the devs plan on creating the battery which can be used to replenish voltage if default given voltage is exhaused.

**Game item creation** - The admin calls the `createGameItem` function to create a new game item of a specific token id. Here, admin can specify the item name, price, uri, its total supply and the user's daily allowance. Admin can also set if the item is transferrable and if the supply is finite.

**Game item purchase, distribution and usage** - To purchase a game item, the user calls the `mint` function passing in the required game item token id and the quantitiy. The function then mints them the item. By calling the `burn` function on behalf of a user, the specified burn addresses (in this case, the VoltageManager contract) can destroy the game item upon usage. If a game item is transferrable, users can distribute it among themselves by calling the `safeTransferFrom` function. Important to note that users have a specific allowance limit for items they can hold. This limit can however be bypassed by transfer. 
 
<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/0630a47e-73c0-4682-ae54-6a127b58a7d0" alt="AiArenaHelper.sol"> 
</p>

<p align="center">
   sLOC - 163
</p>

#### **[VoltageManager.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol)**

- The `VoltageManager` contract is the adapter contract that controls the functions of the battery game item. Here, users can manage the use of their batteries or their voltage.

**Battery Usage** - When a user is low on voltage, they call the `useVoltageBattery` function, which burns a battery from them, in exchange for replenishing their voltage back to max. 

**Voltage usage** - A user or the approved voltage spenders (including the `RankedBattle` contract) can call the `spendVoltage` function to use a certain quantity of the user's remaining voltage. The function also replenishes a user's default voltage if the replenish time has passed.
<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/305a10a4-123a-48a1-8028-d6880bf0aab6" alt="VoltageManager.sol">
</p>

<p align="center">
   sLOC - 47
</p>


#### **[Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol)**

- This is the NRN token contract. It's the ERC20 token native to the protcol used as the protocol's economic system, motivating players and aligning their goals. Players can acquire NRN by buying it on the market place or earning it through rewards. 
- The maximum token supply is about 9 billion, with 200 million being minted directly to the treasury and another 500 to the contributors upon launch.  

**Role Distribution by owner** - The contract owner by calling certain functions can distribute the various contract roles to addresses. The owner can do this by calling the `addMinter`, `addStaker`, `addSpender` and `adjustAdminAccess` functions.

**Airdrop approval and claim** - The admin by calling the `setupAirdrop` function can approve a number of recipients for aidrop. These recipients can now call the `claim` function to be transferred their airdrop due.

**Token minting and burning** - The address with the minter role, the `RankedBattle` address can call the `mint` function to mint a specified amount of tokens to a specified recipient. Users who so wish can call the `burn` and `burnFrom` function to permanently destroy their tokens and remove it from circulation.

**Granting Approvals** - The `RankedBattle` address and any other address with the staker role can call the `approveStaker` function to approve an address to stake on behalf of the user. The same goes for the `FighterFarm` and `GameItem` contracts and addresses with the spender role, upon calling the `approveSpender` function, to approve an address to spend the user's tokens.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/d5f68aa8-adc4-4185-9c10-1f838c2c0eb0" alt="Neuron.sol"> 
</p>

<p align="center">
   sLOC - 92
</p>

***


## **5. Codebase Overview**

- **Audit Information** - For the purpose of the security review, the Ai-Arena codebase consists of eight smart contracts totaling 1271 SLoC. It holds five external imports and four structs. Its core design principle is composition, enabling efficient and flexible integration. It is scheduled to be deployed on the Arbitrum L2 chain, giving each user the freedom to transact in and across all platforms. It depends on the protocol's game server oracle which puts the battle results on chain.

- **Documentation and NatSpec** - The codebase provides important resources in form of the documentation and an overview of the website. While the documentation provides a good enough overview, it appears to be outdated, passing off old information. The contracts are well commented, not strictly to NatSpec but each function was well explained. It made the audit process easier.

- **Error handling and Event Emission** -  Require errors with error strings is used in the codebase. Events are well handled, emitted for important parameter changes, although in some cases, they seemed to not follow the checks-effects-interaction patterns. 

- **Testability** - Test coverage is about 90% which is very gooed. The implemented tests are mosty unit tests which tests the basic functionalities of the functions and helped with basic bugs. The more complex parts of the codebase are not fuzz tested, neither are there any invariant tests.

- **Token Support** - The protocol works with the Fighter ERC721 token, Game Item ERC1155 token and NRN ERC20 tokens. 

- **Attack Vectors** - For the protocol, major points of attack include gaming reward system, losing battles while staking without losing points of tokens, bypassing various protocol and so on.

***

## **6. Centralization Risks** 

Like any protocol that incorporates owner and admin functions including a number of other roles, hence the protocol is fairly decentralized. However, the actions of a malicious central powers can negatively impact the protocol and its users. Some of them include:

- Griefing users by setting `bpsLostPerLoss` to max which causes users to lose all their staked amount upon losing a battle.
- Setting `rankedNrnDistribution` to 0 so that users earn no rewards upon winning battles.
- Setting various important protocol address to zero or wrong values.

***
## **7. Systemic Risks** 

- User activity is very important as the protcol will not really function if users are not very active.
- Malicious users looking for various ways to game the point system while not having much stake to lose.
- The protocol's game server oracle can fail and affect battle report update, hence reward distribution.
- Lack pause protocols in place during emergency market conditions.
- Issues with external dependencies, solidity version bugs and open zeppelin contracts. 
- Issues from other smart contracts, which at best may be contained in the erring contracts, at worst affect the whole protocol.
***
## **8. Recommendations**

- The codebase should be sanitized, and the comments should also be brought up to date to conform with NatSpec. The same goes for the provided documentation. A lot of the information in there are outdated. These need to be fixed.

- While the NRN token is a standard ERC20 token, and is the only ERC20 token interacting with the protocol, using basic transfer methods for is not advisable. The return values are not explicitly checked for failure (only for success), so the protocol still has the problem of transfers failing silently, which in certain situations doesn't restore updated state changes back to default;  Consider using safer methods instead;

- Game items are made with an explicit maximum amount that users are allowed to hold daily, however, this amount doesn't take into consideration that certain items are transferable. Transferring these items can be used to bypass daily limit. Consider deciding if this is intended or taking maximum daily limit into consideration upon transfer.

- Information about the `tokenUri` should also include a chainid in the rarecase of a hardfork. This helps remove the confusion of which owner owns which token on which chain. Important to also note that newly rerolled fighters have an empty string as `tokenUri`, consider setting a default uri for them instead.

- Consider checking also, if a user is using their voltage battery, that a user's replenish time is passed. This saves the users from spending on voltage when they could easily just get their daily replenish voltage.

- In the constructor of most of the contracts, the owner is explicitely set as admin, however, upon ownership transfer, the previous owner is still left as admin, while the new owner doesn't have admin functions. This is not advisable. Consider fixing this, unless, it's by design. 

- The protocol uses the game server as the oracle for updating user records. For this purpose, a fallback oracle will also be of great help in cases of server crash or downtime.

- A proper pause function can be implemented to protect users in cases of turbulent market situations and black swan events. It also helps prevent malicious activities or bugs from causing significant damage while the team investigates the potential issues without affecting users' funds.

- Two step variable and address updates should be implemented. Most of the setter functions implement the changes almost immediately which can be jarring for the contracts/users. Adding these fixes can help protect from mistakes and unexepected behaviour. At the same time, a timelock also helps and gives users/admins time to react quickly to these situtaions. Other sanity checks, max/min values check also protect the users from griefing, intentional or not.

- Testing should be improved, including invariant and fuzzing tests;

- Solidity and OpenZeppelin contract versions should be updated to the latest versions as they provide lots of benefits in security and optimization. It's best to use a specific compiler version;

***
## **9. Conclusions**

- The AI-Arena codebase appears to be fairly well-designed,As is the reason for the audit, the identified risks need to be fixed. Recommended measures should be implemented to protect the protocol from potential attacks. Timely audits and sanitizations should be conducted to keep the codebase fresh and up to date with evolving security times.

***
## **10. Resources**

- [C4 ReadMe](https://code4rena.com/audits/2024-02-ai-arena#top)
- [Documentation](https://docs.aiarena.io/gaming-competition)
- [Website](https://aiarena.io/#/)

### Time spent:
36 hours