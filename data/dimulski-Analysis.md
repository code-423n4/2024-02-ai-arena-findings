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
During this 12-days audit the codebase was tested in various different ways. The first step after cloning the repository was ensuring that all tests provided pass successfully when run. After that I set up a foundry environment in order to write tests using the foundry framework. Then began the phase when I got some high-level overview of the functionality by reading the code. There were some parts of the project that caught my attention. These parts were tested with Foundry POCs. Some of them were false positives, others turned out to be true positives that were reported. The contest's main page does provide some context for the main idea of the AiArena protocol, and points to some useful explanations of the [AiArena ranking system](https://docs.aiarena.io/gaming-competition/ranking-system).
## 3. Architecture recommendations
### Protocol Structure
AiArena is a protocol that falls in the GameFi category. The main goal of the AiArena protocol is to allow users to utalize their understanding of AI in order to create better fighters, by creating better moves and strategies. The protocol has a reward mechanism in which an amount of NRN tokens(the token created by the AiArena team) is distributed to users proportional to the points they have earned by fighting in the arena and winning during the specified round. Rounds according to the protocol team will be bi-weakly, in order for a user to participate the have to own a fighter, and stake some NRN so they can earn points, and compete for rewards. It has to be noted that users may lose part of their stake if they lose more times that they win, or draw. The protocol provides the opurtunity for users to buy different game items, one of them is a battery that allows the user to play more battles once he has drained his voltage for the day. 
### What was unique for me?
 - Unique game design, that allow people to leverage their understanding of AI in order to create a better fighter
## 4. Codebase quality analysis
There are several vulnerabilities within the codebase, but beside that the code is well structured. An improvement to the overall strucutre of the files withing the project is to createa a library directory and put the ``FixedPointMathLib.sol`` and ``Verification.sol`` files in it. It can also be considered to create separate directories to better separate the logic by the contracts responsible for battles and staking, and minting and managing NFTs. The  contracts themselves have been structured in a systematic and ordered manner, which makes it easy to understand how the function calls propagate throughout the system. Some additional suggestions are provided in the Software engineering considerations chapter.
## 5. Mechanism review
### High-level system overview
The Curves protocol falls in the SocialFi category. From a rudimentary point-of-view, SocialFi is the amalgamation of Web3 and Social Media. The idea combines social media and decentralized finance (DeFi) concepts to create, control, and own user-generated content on platforms. Influencers, content producers, and participants that want to govern their data can do so in the SocialFi universe. A new user of the Curves platform can register with his X(formally twitter account), and create an unique curves token, indexed by the user's address. The unique curve tokens that each user can create for himself are identified by an unique address. That address is referred to as the token subject within the Curves protocol. The price for which a curves token for a specific token subject can be bought and sold is determined by formula that increases the price of each next token, as the supply grows, and decreases the price of each token when supply shrinks.
### Chains supported
The contest main page does mention that the protocol will be deployed on an L2, but doesn't specify which one, and if only one. The protocol team has mentioned that they plan to launch on Arbitrum, but also consider other L2 chains, which may be problematic as some chains such as Metis are expected to move to a decentralized sequencer model in Q2 of 2024, which introduces frontrunning possiblities, and the protocol has several instances where such vulnerabilites can be exploited. As per the proposed architecture that is currently being tested by the community [The order of the transactions are determined by the Block Proposer. This is rotated based on the Consensus (POS) layer](https://ethresear.ch/t/pos-sequencer-pool-decentralizing-an-optimistic-rollup/16760/5). Also as the contracts are not upgradable, and in the near future there is the possibility of all L2 following in the steps of Metis and addopting the decentralized sequencer model and introdcuing MEV opurtunities, the frontrunning vulnerabilities within the code should be fixed.

### AiArenaHelper.sol

### FighterFarm.sol

### FighterOps.sol

### GameItems.sol

### MergingPool.sol

### Neuron.sol

### RankedBattle.sol

### StakeAtRisk.sol

### VoltageManager.sol

## 6. Centralization risks
The protocol is not heavily centralized, although there are very importatn parameters that are controled by the manager and owner roles. In the ``Curves.sol`` contract if one of the wallets holding a manager or owner role gets compromised the protocol fees can be set to 100% and the protocolFeeDestination can be set to an address controled by a malicious user, since there is no slippage protections on the ``sellCurvesToken()`` function this can be used to front run  non suspecting users and effectively steal all the ``ETH`` they are supposed to withdraw. Also there are no input validation checks in the setting fees functions, so admins should be extreamly carreful when they set those parameters. There are no timelock functions for actions affecting all the users of the protocol, I would advise the team to consider adding such functions. If an admin account gets compromised, having a timelock function will give the users enough time to withdraw their funds safely if no other actions to protect them can be taken from the team. If the onwer account is compromised a malicious factory contract can be created, that can allow malicious users to mint arbitrary amount of ERC20 tokens that represent certain curve tokens, and later integrate them in the Curves protocol. Consider utalizing multisig wallets such as [Safe](https://safe.global/), in order to better secure the accounts, or implement a DAO and give the owner role to the DAO. Once the vulnerabilities mentioned in the above section in the ``Security.sol`` and ``FeeSplitter.sol`` contracts, are fixed the access control should be sufficient. 

## 7. Security approach of the project
The protocol is missing some edge case tests, as well as other unit tests which don't consider only the expected behaviour of the user. The decision to utalize the foundry framework in order to build the protocol is a great one, although not all benefits of the frameworks are utalized especially when it comes to tests. Consider implementing fuzz(foundry built in fuzzer is a great tool) and invariant tests to better cover edge cases. It is a very good decision that you decided to get your code base audited by a platfrom such a C4, although all auditors put their best effort in finding and submitting all vulnerabilites, we can miss something, creating a bug bounty on platforms such as [immunefi](https://immunefi.com/) creates and additional layer of security. 
## 8. Software engineering considerations
### Modularity and reusability

The contracts withing the protocol exhibit modularity by inheriting from several interfaces and contracts, suggesting components are designed for reuse and extension. This approach aligns with solid software engineering principles, promoting maintainability and scalability.
### Code readability and maintainability

The protocol's naming conventions are not that clear, some improvement may be welcomed, exspecialy in the naming of certain functions which actualy create a cruve token associated with an address, shouldn't have buy in their name. NatSpec is missing the severely decreases the readability and maintainability of the code which is crucial for long-term management and updates.
### Error handling and input validation

The use of custom error messages indicates a proactive approach to error handling, however input validations are missing, especially when setting the protocol fees, this can cause mistakes which results in a loss for the protocol. Input validation should be implemented as it is critical for robustness and user trust.
### Upgradeability

The potential for future upgrades and modifications should be considered, especially in a new and rapidly evolving domain like SocialFi. However, the protocol doesn’t explicitly mention upgrade mechanisms.
### Documentation

Part of the documentation is outdated, and the changes are substantial, having a good  and up to date documentation helps with the long-term management and updates of the contract.









### Time spent:
25 hours

### Time spent:
23 hours