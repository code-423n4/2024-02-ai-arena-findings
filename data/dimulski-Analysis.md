# AiArena analysis report 
## 1. Table of contents
- [Curves analysis report](#aiarena-analysis-report)
  - [1. Table of contents](#1-table-of-contents)
  - [2. Approach taken in evaluating the codebase](#2-approach-taken-in-evaluating-the-codebase)
  - [3. Architecture recommendations](#3-architecture-recommendations)
    - [Protocol Structure](#protocol-structure)
    - [What was unique for me?](#what-was-unique-for-me)
  - [4. Codebase quality analysis](#4-codebase-quality-analysis)
  - [5. Mechanism review](#5-mechanism-review)
    - [High-level system overview](#high-level-system-overview)
    - [Chains supported](#chains-supported)
    - [AiArenaHelper.sol](#aiarenahelpersol)
    - [FighterFarm.sol](#fighterfarmsol)
    - [FighterOps.sol](#fighteropssol)
    - [GameItems.sol](#gameitemssol)
    - [MergingPool.sol](#mergingpoolsol)
    - [Neuron.sol](#neuronsol)
    - [RankedBattle.sol](#rankedbattlesol)
    - [StakeAtRisk.sol](#stakeatrisk)
    - [VoltageManager.sol](#voltagemanagersol)
  - [6. Centralization risks](#6-centralization-risks)
  - [7. Security approach of the Project](#7-security-approach-of-the-project)
  - [8. Software engineering considerations](#8-software-engineering-considerations)
    - [Modularity and reusability](#modularity-and-reusability)
    - [Code readability and maintainability](#code-readability-and-maintainability)
    - [Error handling and input validation](#error-handling-and-input-validation)
    - [Upgradeability](#upgradeability)
    - [Documentation](#documentation)
## 2. Approach taken in evaluating the codebase
During this 12-days audit the codebase was tested in various different ways. The first step after cloning the repository was ensuring that all tests provided pass successfully when run. Then began the phase when I got some high-level overview of the functionality by reading the code. There were some parts of the project that caught my attention. These parts were tested with Foundry POCs. Some of them were false positives, others turned out to be true positives that were reported. The contest's main page does provide some context for the main idea of the AiArena protocol, and points to some useful explanations of the [AiArena ranking system](https://docs.aiarena.io/gaming-competition/ranking-system).
## 3. Architecture recommendations
### Protocol Structure
AiArena is a protocol that falls in the GameFi category. The main goal of the AiArena protocol is to allow users to utilize their understanding of AI in order to create better fighters, by creating better moves and strategies. The protocol has a reward mechanism in which an amount of NRN tokens(the token created by the AiArena team) is distributed to users proportional to the points they have earned by fighting in the arena and winning during the specified round. Rounds according to the protocol team will be bi-weakly, in order for a user to participate the have to own a fighter, and stake some NRN so they can earn points, and compete for rewards. It has to be noted that users may lose part of their stake if they lose more times that they win, or draw. The protocol provides the opportunity for users to buy different game items, one of them is a battery that allows the user to play more battles once he has drained his voltage for the day. 
### What was unique for me?
 - Unique game design, that allow people to leverage their understanding of AI in order to create a better fighter
## 4. Codebase quality analysis
There are several vulnerabilities within the codebase, but beside that the code is well structured. An improvement to the overall structure of the files within the project is to create a a library directory and put the ``FixedPointMathLib.sol``, ``Verification.sol`` and ``FighterOps.sol`` files in it. It can also be considered to create separate directories to better separate the logic by the contracts responsible for battles and staking, and minting and managing NFTs. The  contracts themselves have been structured in a systematic and ordered manner, which makes it easy to understand how the function calls propagate throughout the system, except the ``AiArenaHelper.sol`` contract, which contains some complex logic for determining the physical attributes of the fighters. Some additional suggestions are provided in the Software engineering considerations chapter.

## 5. Mechanism review
### High-level system overview
In order for a user to be able to participate in ranked battles, he has to acquire an NFT fighter first, and NRN tokens. There will be mint passes for users that have qualified based on some parameters, that were not disclosed by the team. There will be airdrops of NRN tokens as well. This is how the first users will be onboarded, later on there will be third party marketplaces where other users can buy fighter NFTs and NRN tokens, in order to participate in ranked battles and thus compete to win more NRN tokens. It is explained by the protocol that knowledge of AI will be beneficial for creating a better fighter, as the model that is equipped to each NFT fighter has a big weight in determining the outcome of a battle(those mechanics are out of scope for the audit and were not disclosed by the protocol team). The main purpose of the protocol is for users to participate in battles, win points, and later on claim the NRN rewards for the round proportionate to their points. When participating in battles users may lose a portion of their stake if they rack up too many defeats. There are different game items that can be utilized in the protocol, the most significant is the battery, which can be used so a user can initialize more battles. Other items can be added later on, that have different additions to the game play, and outcome of battles. Each user have the possibility to win a new NFT each round, if he choses to compete for merging points as well. 

### Chains supported
The contest main page does mention that the protocol will be deployed on an L2, but doesn't specify which one, and if only one. The protocol team has mentioned that they plan to launch on Arbitrum, but also consider other L2 chains, which may be problematic as some chains such as Metis are expected to move to a decentralized sequencer model in Q2 of 2024, which introduces frontrunning possibilities, and the protocol has several instances where such vulnerabilites can be exploited. As per the proposed architecture that is currently being tested by the community [The order of the transactions are determined by the Block Proposer. This is rotated based on the Consensus (POS) layer](https://ethresear.ch/t/pos-sequencer-pool-decentralizing-an-optimistic-rollup/16760/5). Also as the contracts are not upgradable, and in the near future there is the possibility of all L2 following in the steps of Metis and adopting the decentralized sequencer model and introducing MEV opportunities, the frontrunning vulnerabilities within the code should be fixed.

### AiArenaHelper.sol
This contract generates and manages an AI Arena fighters physical attributes. The overall logic is too complex, and it is very hard to follow. There is no true source of randomness implemented in this contract, some of the variable names such as ``rarityRank`` are not well named, and it is not clear from the name what they are expected to represent. There are a couple of vulnerabilities in the ``FighterFarm.sol`` contract, that allow users to manipulate the physical attributes of their NFTs and thus get rarer NFTs, but the root cause is not in this contract. Consider simplifying the logic for picking attributes and utilizing  a true source of randomness such as Chainlink VRF. Also instead of checking in each function if the caller is the owner address, consider implementing modifier, or better yet utilize the Ownable2Step contract from openzeppelin, which comes with a built in modifier, and a safe transfer of ownership, instead of writing your own functions for that ``transferOwnership()``. It is always a good practice to use battle tested libraries. 

### FighterFarm.sol
This is one of the main contracts of the AiArena protocol. It manages the creation, ownership, and redemption of AI Arena Fighter NFTs, including the ability to mint new NFTs from a merging pool or through the redemption of mint passes. Consider adding a function that will allow you to change the cost of the ``rerollCost``, because as time passes by the NRN token can increase in value and the reroll cost may become too big in dollar value, for users to utilize the ``reRoll()`` function. Consider utilizing the Ownable2Step contract from openzeppelin, as in this way you won't have to create your own ``transferOwnership()`` function, as well as having to check in each function if the ``msg.sender`` is the owner, as the Ownable2Step contacts comes with a built in modifier. There are a couple of vulnerabilities within this contract, allowing users to mint NFTs whit whatever attributes they desire, as well as transfer NFTs, which shouldn't be transferable. However once those vulnerabilities are fixed the contract should function as expected.

### FighterOps.sol
Is a simple library that contains two structs and an even, as well as a couple of view functions, there is not much that can be said about this library.

### GameItems.sol
This contract represents a collection of game items used in AI Arena. During the audit of the AiArena protocol the only item that is known to be utilized for sure is the battery (users can buy batteries, in order to recharge their voltage, and initialize more battles once the deplete their daily allowance). Consider utilizing the Ownable2Step contract from openzeppelin, as in this way you won't have to create your own ``transferOwnership()`` function, as well as having to check in each function if the ``msg.sender`` is the owner, as the Ownable2Step contacts comes with a built in modifier. Also instead of checking if the caller of a specific function is an admin, consider implementing a modifier as there are a couple of instances where such checks are performed. There is a vulnerability allowing items that shouldn't be transferable to be transferred but once this is fixed the contract should function as expected. 

### MergingPool.sol
This contract allows users to potentially earn a new fighter NFT. When a user wins a battle he can choose to have some of his points allocated to the ``MergignPool.sol`` contract. The main page of the contest mentions that there will be a raffle to determine the winners of each round, but the ``pickWinner()``function is called by an admin, and there is no randomness that can be verified on chain that is used to decide who the winners are. Also there is no clear use case of the points won by a user, as from the contract code it doesn't seem they have any weight on determining the winner. I would recommend to either utilize a Chainlink VRF as a source of randomness or take into account how many points the users have won in that round, when determining the winners. Implement it in a way that can easily be proven on chain, that will create more trust in the user base of the protocol. Other than that the contract should function as expected. 

### Neuron.sol
The Neuron token is used for various functions within the platform, including staking, and rewards. Granting mint, approve and burn roles to anyone the owner decides to, is a dangerous practice, because if the owner address gets compromised all the balances will be stolen. Consider utilizing the ERC2612 instead, more safety measures are discussed in the Centralization risks chapter. The way the ``AccessControl.sol`` is implemented in the ``Neuron.sol`` contract is not according to the best practices, as ``_setupRole()`` is called outside of the constructor consider using ``grantRole()`` to follow best practices. Consider utilizing the Ownable2Step contract from openzeppelin, as in this way you won't have to create your own ``transferOwnership()`` function, as well as having to check in each function if the ``msg.sender`` is the owner, as the Ownable2Step contacts comes with a built in modifier. Also instead of checking if the caller of a specific function is an admin, consider implementing a modifier as there are a couple of instances where such checks are performed. Other than that this contract should function properly. 

### RankedBattle.sol
This contract provides functionality for staking NRN tokens on fighters, tracking battle records, calculating and distributing rewards based on battle outcomes and staked amounts, and allowing claiming of accumulated rewards. Admins should be extremely careful when the call the ``setBpsLostPerLoss()`` function, because if it is called during an ongoing round it will be problemtaic for the rewards at stake calculations for users, for those that lost a substantial amount, it will be beneficial if bpsLostPerLoss is increased, as it will require less wins to get back the full amount. Consider adding a pause functionality to this contract, as this way it will be much safer to start a new round, set new bpsLostPerLoss, or modify the addresses of some of the contracts the ``RankedBattle.sol`` contract interacts with. Especially the ``StakeAtRisk.sol`` contract. Consider utilizing the Ownable2Step contract from openzeppelin, as in this way you won't have to create your own ``transferOwnership()`` function, as well as having to check in each function if the ``msg.sender`` is the owner, as the Ownable2Step contacts comes with a built in modifier. Also instead of checking if the caller of a specific function is an admin, consider implementing a modifier as there are a couple of instances where such checks are performed. There are a couple of vulnerabilities within the contract regarding, malicious users utilizing insufficient validations in order to win more points, once those vulnerabilities are fixed this contract should function properly. 

### StakeAtRisk.sol
This contract allows the ``RankedBattle.sol`` contract to manage the staking of NRN tokens at risk during battles. Consider adding a function to change the ``treasuryAddress`` if it gets compromised, this will require an admin or owner access control as well. There is not much else that can be said about this contract. 

### VoltageManager.sol
This contract allows the management of voltage for game items and provides functions for using and replenishing voltage. Consider utilizing the Ownable2Step contract from openzeppelin, as in this way you won't have to create your own ``transferOwnership()`` function, as well as having to check in each function if the ``msg.sender`` is the owner, as the Ownable2Step contacts comes with a built in modifier. There is some vulnerabilities that may arise if the protocol is deployed on a chain that allows for frontrunning, as the ``spendVoltage()`` function can be abused by a malicious user in order to drain his voltage so he can revert battles, the outcome of which he doesn't like. There is not much else that can be said about this contract. 

## 6. Centralization risks
The protocol is heavily centralized. All important parameters can be changed either by the owner or by an admin. Combining it with the fact that there are no input validation this is a real centralization risk. In the ``FighterFarm.sol`` contract if the ``_delegatedAddress`` is compromised, there is no way to change that address, and now an infinite number of NFTs can be minted by malicious users. Consider adding a function that will allow you to change this address in the scenario it gets compromised, also consider a timelock function. There are no timelock functions for actions affecting all the users of the protocol, I would advise the team to consider adding such functions. If an admin account gets compromised, having a timelock function will give the users enough time to withdraw their funds safely if no other actions to protect them can be taken from the team. If the owner account is compromised all contract addresses can be changed as well as all parameters, the biggest concern is the way the ``Neuron.sol`` contract is implemented. A burn, approve and mint roles can be granted by the owner to anyone. If the owner gets compromised all of the NRN balances of all the users can be drained. Consider utilizing multisig wallets such as [Safe](https://safe.global/), in order to better secure the accounts, or implement a DAO and give the owner role to the DAO. As pointed above consider utilizing openzeppelin Owanble2Step contract, for safe ownership transfer. 

## 7. Security approach of the project
The protocol is missing some edge case tests, as well as other unit tests which don't consider only the expected behavior of the user. The decision to utilize the foundry framework in order to build the protocol is a great one, although not all benefits of the frameworks are utilized especially when it comes to tests. Consider implementing fuzz(foundry built in fuzzer is a great tool) and invariant tests to better cover edge cases. It is a very good decision that you decided to get your code base audited by a platform such a C4, although all auditors put their best effort in finding and submitting all vulnerabilities, we can miss something, creating a bug bounty on platforms such as [immunefi](https://immunefi.com/) creates and additional layer of security. 
## 8. Software engineering considerations
### Modularity and reusability

The contracts within the protocol exhibit modularity by inheriting from several interfaces and contracts, suggesting components are designed for reuse and extension. This approach aligns with solid software engineering principles, promoting maintainability and scalability.
### Code readability and maintainability

Most of the protocol's naming conventions are clear, with few exceptions mentioned above, some improvement may be welcomed. NatSpec is well defines which is really a good practice beneficial for the readability and maintainability of the code which is crucial for long-term management and updates. Although there are some instance where @return and @param tags are missing, I would recommend to add those.
### Error handling and input validation

It is a good practice to utilize custom error messages this indicates a proactive approach to error handling, input validations are also missing, especially when setting the bpsLostPerLoss, and no address(0) checks,  this can cause mistakes which results in a loss for the protocol. Input validations as well as custom error messages should be implemented as it is critical for robustness and user trust.
### Upgradeability

The potential for future upgrades and modifications should be considered, especially when a lot of L2s, have plans about moving towards decentralized sequencers, which may introduce frontrunning vulnerabilities. However, the protocol doesn’t explicitly mention upgrade mechanisms.
### Documentation

Part of the documentation is outdated, and the changes are substantial, having a good and up to date documentation helps with the long-term management and updates of the contract.

### Time spent:
25 hours



### Time spent:
25 hours