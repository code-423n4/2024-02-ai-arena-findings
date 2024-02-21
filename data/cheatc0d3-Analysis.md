# AI Arena Security Analysis Report

![Pic](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FXEnWzzABSLk.0&w=256&q=75)


## Overview of the AI Arena Project

AI Arena represents a pioneering convergence of blockchain technology, competitive gaming, and artificial intelligence. It is a player-versus-player (PvP) platform fighting game where the combatants are AI entities trained by human players. Leveraging the Ethereum blockchain, AI Arena introduces a unique ecosystem where these AI fighters are tokenized as NFTs through the FighterFarm.sol smart contract, blending the thrill of gaming with the world of digital collectibles and decentralized finance.

**Core Components and Mechanisms:**

1. **Fighter NFTs**: At the heart of AI Arena are the fighter NFTs, each possessing unique characteristics such as physical attributes, generation, weight, element, and fighter type. These traits not only define the fighter's visual appeal but also their combat capabilities and special abilities in the arena.

2. **$NRN Token**: AI Arena's native ERC20 token, as outlined in the Neuron.sol smart contract, serves multiple roles within the ecosystem, including transactional currency, staking for rewards, and governance. The deployment process meticulously assigns roles such as MINTER and STAKER to the RankedBattle.sol contract and the SPENDER role to FighterFarm.sol and GameItems.sol contracts to streamline in-game transactions and reward distribution.

3. **Ranked Battles**: Central to the AI Arena experience, ranked battles allow players to pit their AI-trained fighters against one another. Success in these competitions is rewarded with $NRN tokens, with the amount staked influencing potential rewards. However, poor performance could lead to partial stake loss, adding a layer of strategy to participant engagement.

4. **Staking Mechanism**: To ensure fairness and encourage strategic play, AI Arena employs a unique calculation for rewards based on the square root of the amount staked, termed as the stakingFactor. This approach aims to level the playing field by moderating the influence of large stakes on the competition's outcome.

5. **Voltage System**: Players manage a "voltage" resource that depletes with each ranked battle and replenishes over time. This system necessitates strategic resource management, with players needing to balance their engagement against the availability of voltage. For immediate replenishment, players can purchase batteries via the GameItems.sol smart contract, further integrating the game's mechanics with its token economy.

6. **Off-Chain Game Logic**: AI Arena's core gameplay and logic, including the determination of ranked match outcomes, are processed off-chain through dedicated game servers. These servers act as oracles, bridging the in-game events with the blockchain to maintain integrity and fairness in match results and rewards distribution.

## Evaluation Approach

In evaluating the AI Arena project, I focused on understanding the architecture and functionalities presented across the various Solidity contracts. My approach involved dissecting the main components of the ecosystem, including how fighters are managed, battles are conducted, and how the in-game economy functions through the use of an ERC20 token system. Here's a closer look at my thought process and the key areas I considered:

### Architecture and Core Interactions
I started by examining how the contracts interact with each other, identifying the roles of `FighterFarm`, `RankedBattle`, `Neuron`, `GameItems`, `VoltageManager`, `MergingPool`, and `StakeAtRisk` in the ecosystem. My aim was to understand how these components support gameplay, from managing fighters and conducting battles to handling the game's economy and unique features like voltage and staking.

### Security and Design
Given the complexity of interactions and the potential risks associated with smart contracts, I paid special attention to security mechanisms, particularly role-based access controls. I evaluated how these roles are managed and the implications for game integrity and security. Additionally, I considered the AI Arena documentation in my review and checked for correctness in implementation.


## System Overview

### Scope

The following advanced table provides an in-depth overview of these contracts, highlighting their source lines of code (SLOC), purpose, key functionalities, and utilized libraries:

| File                | SLOC | Purpose                                                                                                     | Key Functionalities                                                                                                                                                                                                                                           | Libraries Used                                                                                   |
|---------------------|------|-------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| **AiArenaHelper.sol** | 95   | Physical Attributes Management                                                                              | - Generates physical attributes for fighters - Maintains attribute probabilities and DNA divisors for generation-based fighter customization                                                                                                               | - OpenZeppelin Contracts - FighterOps for struct definitions                                  |
| **FighterFarm.sol**   | 327  | NFT Management and Minting                                                                                  | - Tokenization and management of fighter NFTs - Ownership tracking and transfer - Minting through merging pool or mint passes - Attributes assignment (physical, generation, weight, element, type, model data)                                      | - OpenZeppelin ERC721, Enumerable - FighterOps for NFT attributes - Verification for signatures |
| **FighterOps.sol**    | 74   | Fighter Data Handling                                                                                       | - Defines the Fighter struct including all in-game attributes - Provides methods for viewing fighter information                                                                                                                                           |                                                                                               |
| **GameItems.sol**     | 163  | In-Game Items and Utilities Management                                                                      | - ERC1155 implementation for game items - Handles in-game purchases with $NRN tokens - Manages finite supply items and daily allowances - Voltage batteries for ranked battle participation                                                          | - OpenZeppelin ERC1155                                                                         |
| **MergingPool.sol**   | 110  | Reward Distribution and Fighter NFT Earning                                                                  | - Tracks fighter performance and points in competitions - Selects winners per competition period and rewards with fighter NFTs - Manages claiming process for earned rewards                                                                            |                                                                                              |
| **Neuron.sol**        | 92   | Platform Currency (ERC20 Token)                                                                              | - ERC20 token for transactions within AI Arena - Facilitates staking, governance, and reward mechanisms - Grants MINTER and SPENDER roles to specific contracts for controlled token distribution and spending                                         | - OpenZeppelin ERC20, AccessControl                                                            |
| **RankedBattle.sol**  | 300  | Gameplay Mechanics and Staking                                                                               | - Organizes ranked battles and tracks outcomes - Manages staking of $NRN tokens on fighters - Calculates rewards based on battle outcomes and stakes - Interfaces with off-chain logic for match results                                              | - FixedPointMathLib for staking calculations                                                   |
| **StakeAtRisk.sol**   | 63   | Risk Management for Staked Tokens                                                                            | - Tracks $NRN tokens at risk from poor performance<br>- Allows reclaiming of at-risk stakes for winning fighters<br>- Adjusts stakes based on battle outcomes                                                                                                 |                                                                                              |
| **VoltageManager.sol**| 47   | Player Participation and Resource Management                                                                 | - Manages voltage as a resource for initiating battles<br>- Replenishes voltage through natural regeneration or item purchases<br>- Integrates with GameItems.sol for voltage battery purchases<br>- Controls voltage spending for ranked battle participation |                                                                                              |

## Architecture and Activity Diagrams

- https://i2.paste.pics/d2e5e85de67f98a6bb8fd8f3ceacf74d.png?rand=nkapCMhqKs

- https://i2.paste.pics/a0a33ddbe79d43b88b9bb010dc6706c4.png?rand=PiIH4MjVA0

## Centralization Risks Review

In my analysis of the AI Arena project, I paid particular attention to the centralization risks inherent in the system's design and governance. These risks are especially pertinent given the project's reliance on smart contracts and blockchain technology, which ideally aim for decentralization to enhance security and trust. Here are the centralization risks I identified and their potential impact on the project:

### Admin-Controlled Functions and Governance
I noticed that the project incorporates admin-controlled functions and governance mechanisms, which, while necessary for management and updates, concentrate significant power in the hands of a few. This centralization can lead to vulnerabilities if these privileged accounts are compromised, potentially allowing unauthorized changes to game mechanics or misappropriation of funds. It also raises concerns about trust, as the community might question the fairness and transparency of decision-making processes.

### Economic Centralization
The project's economic model, particularly the distribution and management of the in-game currency (`Neuron` tokens), staking mechanisms, and reward systems, presents another centralization risk. If the control over these economic levers is too centralized, it might lead to imbalances in the game's economy, favoritism, or manipulation. This centralization could deter players who feel the system is rigged or unfair.


## Access Control Roles

The project employs a role-based access control (RBAC) system, which is a best practice in smart contract development for defining clear permissions and responsibilities. Specific roles identified include:

- **Owner/Admin**: Typically has the highest level of control, capable of performing critical functions such as updating contract parameters, managing other roles, or even pausing certain aspects of the contract in emergencies.
- **Minter Role**: Assigned to addresses authorized to mint new tokens. In the context of the `Neuron` ERC20 token, this role is crucial for controlling the token supply and ensuring that minting does not exceed predefined limits.
- **Spender Role**: Allows addresses to spend tokens on behalf of the contract, crucial for functions that involve token transfers as part of the game mechanics.
- **Staker Role**: Specific to contracts that manage staking mechanisms, ensuring that only authorized contracts or addresses can interact with staking functions.

### Role Assignment and Management
The contracts appear to employ a centralized approach for assigning and managing roles, typically vested in the owner or admin address. This approach is effective for straightforward role management but raises concerns about single points of failure and centralization risks.

### Security of Role-Based Functions
Functions restricted by roles are critical to the system's integrity, such as token minting, rewards distribution, and access to sensitive game mechanics. The contracts effectively use modifiers to restrict access to these functions, mitigating unauthorized access risks.

### Transferability of Roles
The ability to transfer roles, especially the owner or admin role, is present, which is important for adaptability and potential ownership transfers. However, this also necessitates robust security measures to prevent unauthorized transfers like two step transfers.

### Access Controlled Functions

FighterFarm contract:
- addStaker: Only the owner address can call this to add a new staker address
- instantiateAIArenaHelperContract: Only the owner address can call this 
- instantiateMintpassContract: Only the owner address can call this
- instantiateNeuronContract: Only the owner address can call this
- setMergingPoolAddress: Only the owner address can call this
- setTokenURI: Only the delegated address can call this

MergingPool contract: 
- transferOwnership: Only the owner address can call this
- adjustAdminAccess: Only the owner address can call this  
- updateWinnersPerPeriod: Only admins can call this
- pickWinner: Only admins can call this

Neuron contract:
- transferOwnership: Only the owner address can call this
- addMinter: Only the owner address can call this
- addStaker: Only the owner address can call this  
- addSpender: Only the owner address can call this
- adjustAdminAccess: Only the owner address can call this
- setupAirdrop: Only admins can call this

RankedBattle contract:
- transferOwnership: Only the owner address can call this
- adjustAdminAccess: Only the owner address can call this
- setGameServerAddress: Only the owner address can call this
- setStakeAtRiskAddress: Only the owner address can call this  
- instantiateNeuronContract: Only the owner address can call this
- instantiateMergingPoolContract: Only the owner address can call this
- setRankedNrnDistribution: Only admins can call this
- setBpsLostPerLoss: Only admins can call this 
- setNewRound: Only admins can call this

GameItems contract:
- transferOwnership: Only the owner address can call this
- adjustAdminAccess: Only the owner address can call this    
- adjustTransferability: Only the owner address can call this
- setAllowedBurningAddresses: Only admins can call this
- setTokenURI: Only admins can call this
- createGameItem: Only admins can call this

VoltageManager contract:
- transferOwnership: Only the owner address can call this
- adjustAdminAccess: Only the owner address can call this
- adjustAllowedVoltageSpenders: Only admins can call this

StakeAtRisk contract: 
- setNewRound: Only RankedBattle contract can call this
- reclaimNRN: Only RankedBattle contract can call this
- updateAtRiskRecords: Only RankedBattle contract can call this

## Test Coverage

| File                      | % Lines          | % Statements     | % Branches      | % Funcs          |
|---------------------------|------------------|------------------|-----------------|------------------|
| script/Deployer.s.sol     | 0.00% (0/41)     | 0.00% (0/52)     | 100.00% (0/0)   | 0.00% (0/2)      |
| src/AAMintPass.sol        | 97.14% (34/35)   | 97.50% (39/40)   | 13.64% (3/22)   | 91.67% (11/12)   |
| src/AiArenaHelper.sol     | 94.87% (37/39)   | 96.77% (60/62)   | 33.33% (6/18)   | 100.00% (7/7)    |
| src/FighterFarm.sol       | 97.67% (84/86)   | 97.06% (99/102)  | 25.00% (12/48)  | 92.00% (23/25)   |
| src/FighterOps.sol        | 100.00% (3/3)    | 100.00% (3/3)    | 100.00% (0/0)   | 100.00% (3/3)    |
| src/FixedPointMathLib.sol | 0.00% (0/40)     | 0.00% (0/34)     | 0.00% (0/10)    | 12.50% (1/8)     |
| src/GameItems.sol         | 96.23% (51/53)   | 94.34% (50/53)   | 42.50% (17/40)  | 100.00% (16/16)  |
| src/MergingPool.sol       | 95.74% (45/47)   | 96.72% (59/61)   | 25.00% (5/20)   | 100.00% (8/8)    |
| src/Neuron.sol            | 96.67% (29/30)   | 97.06% (33/34)   | 0.00% (0/26)    | 100.00% (12/12)  |
| src/RankedBattle.sol      | 88.64% (117/132) | 89.29% (125/140) | 38.37% (33/86)  | 100.00% (20/20)  |
| src/StakeAtRisk.sol       | 100.00% (19/19)  | 100.00% (20/20)  | 33.33% (4/12)   | 100.00% (5/5)    |
| src/Verification.sol      | 70.00% (7/10)    | 76.92% (10/13)   | 0.00% (0/2)     | 100.00% (1/1)    |
| src/VoltageManager.sol    | 100.00% (18/18)  | 100.00% (18/18)  | 35.71% (5/14)   | 100.00% (6/6)    |
| Total                     | 80.29% (444/553) | 81.65% (516/632) | 28.52% (85/298) | 90.40% (113/125) |



## Codebase Review

| Category | Review |
|-|-|
| Code Maintainability | The code is well modularized into separate contracts for different functionality, which improves maintainability. |   
| Errors | Custom errors could be defined and used for more precise error handling. Using require with custom error messages is good but doesn't allow handling errors granularly. |
| Events | Events are used appropriately throughout to log key actions in an extensible way. |
| Code Readability | Overall very readable code - uses descriptive names, good modularization, and comments appropriately. |
| Gas Efficiency | Mapping structs are used instead of arrays in various places which is more gas efficient. Further gas optimizations could potentially be made through use of libraries instead of inheritence where applicable. |
| Code Style | Excellent adherence to Solidity style guide - easy to read and understand. |
| Code Upgradability | Lack of proxy contracts or explicit storage layout for core contracts could make upgrades difficult after launch. |
| Code Complexity | There are many interconnected contracts and complex game mechanics. More prone to bugs and hard to fully test boundaries. |

## Findings

| Severity | Findings          |
|----------|-------------------|
| Critical | 0                 |
| High     | 0                 |
| Medium   | 0                 |
| Low      | 4                 |
| NC      | 1                 |
| Gas      | 3                 |

## Time Spent 

|            |            |
|------------|------------|
| Start Date | 09.02.2024 |
| End Date   | 21.02.2024 |
| Days spent | 10 Days     |
| Hours      | 60h        |

### Time spent:
60 hours