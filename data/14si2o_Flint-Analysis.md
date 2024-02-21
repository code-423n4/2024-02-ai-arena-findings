# Analysis of AiArena

![AiArena](https://github.com/Kraelog/images/blob/e9feab7ddb73fb9fb074a667cd550572619baed7/AiArena.jpg)

## 1. Description AiArena


AI Arena is a PvP platform fighting game where players can develop and use AI models to let charachters battle against each other. In this way it functions both as game and as an AI education platform.  
  
In the web3 version of the game, which is currently under audit, the characters are tokenised into NFT fighters with each their unique attributes and loaded up with the AI model provided by the players. Players then stake NRN and battle to gain points while risking to lose a tiny fraction of their stake if they lose a battle. After a round prizes are distributed and a semi-random draw is done for the rights to mint a new fighter. The NFT's can also be traded freely, which opens up an interesting secondary market.    


## 2. Approach taken for this contest

### Time spent on this audit: 9 days

**Day 1:**
 - Reading the documentation
 - First pass through the code with a bottom to top approach
 - Adding @audit-info tags anywhere a question came up

**Day 2-3:**
 - Drawing in Excalidraw:
   - High-level Architecture of the protocol
   - Function diagrams of every contract
   - User Flows 

**Day 4-5:**
 - Second pass through the code
 - Line-by-line review of every contract
 - Adding @audit-issue tags whenever I see something wrong
 - Start writing Notes.md with preliminary findings. 

**Day 6-8:**
  - Foundry Testing of all @audit-issue tags
  - Writing up reports & creating a POC when I have confirmed a finding
  - Making sure that all tags were properly treated
  - Writing QA report
    
**Day 9:**
  - Writing Analysis report

## 3. Architecture

![Architecture](https://github.com/Kraelog/images/blob/28d3aad42faca4c8453305d3bff08cdaf5a1e697/Architecture.png)

The core of the protocol consist of two contracts,`FighterFarm.sol` and `RankedBattle.sol`.   

`FighterFarm` is the NFT Factory, it contains the various ways of minting a Fighter NFT and also the semi-random calculations used to obtain the various attributes of the NFTs.  

`RankedBattle` is the accounting module of the system. It contains the functionality for staking, unstaking and the claiming of rewards. It also receives the outcome of NFT battles from the gameserver, after which it calculates the lossess and gains of the players and tries to ensure that all state variables are correctly updated.     

## 4. Main contracts and functionality

For each contract I have provided a description, an overview of the most important functions available to normal users and a diagram to visualise internal and external calls. 

**RankedBattle.sol**: This is the accounting contract, which handles the staking, battle records and reward calculation/distribution based upon the battle outcomes.
  - **stakeNRN**: This function allows a user to stake NRN on a specific fighter NFT. 
  - **unstakeNRN**: This allows the reverse, the unstaking of NRN from a specific fighter NFT.
  - **claimNRN**: A user can claim NRN from past rounds.
  - **getUnclaimedNRN**: This returns the unclaimed NRN tokens for a specified address.
     
![RankedBattle](https://github.com/Kraelog/images/blob/28d3aad42faca4c8453305d3bff08cdaf5a1e697/RankedBattle.png)


**FighterFarm.sol**: This contract functions as the NFT Factory and is the central point of management for the creation, ownership and redemption of Fighter NFTs.  
  - **claimFighters**: This enables users to claim a pre-determined number of fighters.  
  - **redeemMintPass**: A user can exchange n number of mint passess for fighter NFTs.  
  - **updateModel**: The owner of an NFT Fighter can change the machine learning model used by the fighter through this function.   
  - **(safe)transferFrom** : These 2 functions allow a user to transfer the ownership off an NFT fighter.  
  - **reroll**: A user can re-roll an existing fighter NFT to have new random physical attributes.  

![FighterFarm](https://github.com/Kraelog/images/blob/28d3aad42faca4c8453305d3bff08cdaf5a1e697/FighterFarm.png)



**AiArenaHelper.sol**: This contract handles the creation and management of the attributes of the NFT fighters.  
  - **createPhysicalAttributes**: This functions returns randomised physical attributes which can be used in the minting of the fighter NFT.  
  - **getAttributeProbalities**: This simply returns the attribute probabilities for a given generation and attribute.  
  - **dnaToIndex**: The function converts dna and rarity rank into a attribute probability index.  


![AiArenaHelper](https://github.com/Kraelog/images/blob/4403cc49daecbc94c74312f0e27e71cc1b43d406/AiArenaHelper.png)


**FighterOps.sol**: This library is used for creating and managing Fighter NFTs in the AI Arena game
  - **getFighterAttributes**: This returns the fighter attributes from the **Fighter** struct.
  - **viewFighterInfo**: This returns all relevant fighter information.
![FighterOps](https://github.com/Kraelog/images/blob/28d3aad42faca4c8453305d3bff08cdaf5a1e697/FighterOps.png)

**GameItems.sol**: This contract provides a range of game items that can be used by players in the game. 
  - **mint**: A user can spend NRN to mint and receive a game item.
  - **getAllowanceRemaining**: This function informs a user the remaining number of items that can still be minted today. 
  - **remainingSupply**: This returns the remaining quota for a specific game item. 
  - **safeTransferFrom**: This transfers a game item to a new owner.  

![GameItems](https://github.com/Kraelog/images/blob/28d3aad42faca4c8453305d3bff08cdaf5a1e697/GameItems.png)

**MergingPool.sol**: This contract allows users to earn a new fighter NFT assuming they have won rounds. 
  - **claimRewards**: The function checks the rounds and mints a new NFT for each round that the user won. 
  - **getUnclaimRewards**: This returns the amount of unclaimed rewards for the addres inputted. 
  - **getFighterPoints**: This rerurns an array with the points for all fighters upto a **maxId**.
    
    ![MergingPool](https://github.com/Kraelog/images/blob/28d3aad42faca4c8453305d3bff08cdaf5a1e697/MergingPool.png)


**Neuron.sol**: The ERC20 Neuron token contract which is used for the in game economy.
  - **claim**: A user can call this function to claim a specific amount of tokens from the treasury.
  - **burn**: This burns the specified amount of tokens from the caller's address.
  - **burnFrom**: This burns the specified amount of tokens from the account address. 

    ![Neuron](https://github.com/Kraelog/images/blob/88d717bf3128f66d0f628a05172b88e87d16894f/Neuron.png)


**StakeAtRisk.sol**: This contract manages the staked NRN that are at risk during a round of competition. It is mainly used by **RankedBattle**.
  - **getStakeAtRisk**: This returns the stake at risk for a specific fighter NFT in the current round. 

    ![StakeAtRisk](https://github.com/Kraelog/images/blob/88d717bf3128f66d0f628a05172b88e87d16894f/StakeAtRisk.png)

**VoltageManager.sol**: This contract manages the voltage system for AI Arena.
  - **useVoltageBattery**: This allows a user to use a voltage battery to replenish voltage.
  - **spendVoltage**: This functions spends the voltage.
    
![VoltageManager](https://github.com/Kraelog/images/blob/88d717bf3128f66d0f628a05172b88e87d16894f/VoltageManager.png)

## 5. User flows

The user flows are organised in chronological order. 

#### 1. Obtaining a Fighter NFT  

  ![Fighter NFT](https://github.com/Kraelog/images/blob/88d717bf3128f66d0f628a05172b88e87d16894f/UserFlowObtainAFighter.png)

The first action of anyone who wants to participate in AiArena is to obtain a Fighter NFT. At launch, the `redeemMintPass()` function will be the sole way to mint a fighter NFT. After some time, the protocol will pick winners who can claim new fighters from the mergingPool and/or distribute signatures that can be redeemed for a new Fighter. Additionally, it is possible to buy a NFT Fighter on the secondary market the moment the protocol goes live.


#### 2. Staking & Unstaking  

  ![Staking/Unstaking](https://github.com/Kraelog/images/blob/88d717bf3128f66d0f628a05172b88e87d16894f/UserFlowStakingAndUnstaking.png)

If a player wishes to do battle, he needs to stake some NRN. If he no longer wishes to risk his NRN, he can unstake.  
It is important to note that in its current iteration the staking and unstaking is prone to malicious misuse. 
- There is no minimum stake required, so it is possible to deposit dust so small it cannot be lost, but which still allows a player to acquire points and prizes.
- In the other sense, if a player has lost some of his NRN due to bad play, he can unstake but still continue to play risk-free in order to regain some of his lost NRN. These unhappy/malicious paths have not been sufficiently explored. 

#### 3. Battle  

   ![Battle](https://github.com/Kraelog/images/blob/88d717bf3128f66d0f628a05172b88e87d16894f/UserFlowBattle.png)
   
The battle mechanism can be called the pivotal part of the entire system since the success of the entire protocol rests upon its ability to correctly distribute gains and losses to players. The functions receive the battle outcome from the off-chain gameserver and the entire decision tree is run for each of the participants of the battle. The system works correctly when a player uses a brand new fighter NFT.  

Yet the consequences of selling and buying fighters on the secondary market have not been fully taken into account. There are multiple mechanisms where the points per fighter are conflated with the points per player, which causes unintended negative consequences for the players. 
Some examples of these (which I submitted as findings):  
- If a player sells a fighter with a positive stakeAtRisk, the new owner can drain the lost stake of the previous owner if he plays in the same round. Without any need for staking.  
- The balance of mergingPool points is not stored on the player address who won the points. It is only stored on the fighter so if the fighter is sold, the player loses his entire balance.   


#### 4. Claiming NRN  

  ![Claiming NRN](https://github.com/Kraelog/images/blob/88d717bf3128f66d0f628a05172b88e87d16894f/UserFlowClaimNRN.png)
   
After a round has been concluded, players can claim rewards. Their rewards will be calculated by comparing their points with the total points generated in the round. 
 


## 6. Code Quality / Test Quality

### Code

Overall, the quality of the code is very good. There are some points which can be improved:  
- Some require statements are incomplete and allow bad input to pass
- Floating pragma's should be avoided
- Consider using constants instead of magic numbers
- Events do not make use of indexed fields
- require statements should have descriptive messages
- NatSpec is missing various tags throughout the protocol

### Testing

There clearly has been an effort to achieve near 100% test coverage, which is very good. However, the tests only explore the expected happy paths and do not consider unhappy paths and/or random input.   

An example of this is the `reRoll(uint8 tokenId, uint8 fighterType)` function.   
There are 2 tests for this function:
- `testReroll()` which tests the happy path.
- `testRerollRevertWhenLimitExceeded()` which checks that the maxReroll is correctly applied.

It would seem everything is working fine. But if a user calls reroll with the wrong fightertype, he instantly rerolls his NFT to the highest rarity for all attributes. A critical error which would destroy the value of all AiArena NFTs and one that would be instantly found with the most basic of fuzz tests. 
Therefore I would recommend that the testing should be expanded to at least include basic fuzzing.  

## 7. Risks

### Centralisation

#### 1. Arrays of Admins.

In various contracts the owner can set an entire array of Admins, who have far reaching power. The greater the amounts of admins, the greater the chance that a malicious actor can sneak in or that a normal actor turns malicious. Therefore it is recommended to limit the amount of actors with admin power to the utmost, since they have the capability to severely damage the protocol.     

Admin powers in each contract:
- RankedBattle:
  - setRankedNRNDistribution
  - setBpsLostPerLoss
  - setNewRound
- GameItems:
  - setAllowedBurningAddresses
  - createGameItem
- MergingPool:
  - updateWinnersPerPeriod
  - pickWinner
- VoltageManager:
  - adjustAllowedVoltageSpenders
 
#### 2. One-Step ownership change.

For all the contracts, the change in ownership happens immediately in one step. This carries a very high degree of risk since inputting the wrong address or having the owner hacked and hacker taking the ownership of the protocol cannot be blocked. It is recommended to implement a two-step system whereby a or multiple admins would have to approve the change in ownership. 

### Systemic

#### 1. Frontrunning & Arbitrum Decentralisation

The protocol in its current state would be severily damaged if players could frontrun a loss by unstaking. Currently this is impossible since the sequencer of arbitrum is still centralised. However, the decentralisation is planned and will happen. The protocol should invest sufficient time and effort to adapt the functionality of the protocol to make frontrunning on a decentralised Arbitrum (or other chains) as difficult as possible. 

#### 2. Lack of Upgradability 

Even though immutability is regarded as one of the hallmarks strong points of web3, most protocols have included some form of upgradabilty in order to deal with the inevitable bugs and flaws that will show themselves over time. No project has perfect code.  

For AiArena, the developers have not chosen any of the common upgradabilty proxy patterns but instead they have provided functions in each contract to change the address of the related contracts. If for example, a flaw was found in the GameItems contract, a new contract would be deployed and the admins can set the gameItemsContractInstance to the new address in every contract that requires. So not a system of upgrades, but more of a replacement. If this is used for contracts that hold state (FighterFarm/RankedBattle) however, the redeployment would effectively mean wiping out the entire protocol and starting with a blank slate. Which would significant reputation damage and the risk that users will not risk staking in the protocol. 


## 8. Time spent

58 Hours


### Time spent:
58 hours