## 1. Introduction

This is basic introduction of what AI Arena is. AI Arena is PvP platform where human-trained AI fighters are tokenized using FighterFarm.sol. Each fighter NFT has unique attributes affecting appearance and battle prowess. Players compete in ranked battles to earn $NRN tokens via RankedBattle.sol, with rewards managed by Neuron.sol. Staking tokens is key, with potential losses for poor performance, while voltage management is crucial for participation, replenished daily or via GameItems.sol purchases.

## 2. Architectural Improvement

I will lay out best architectural improvement by contracts in sequence. There are Nine contracts in scope.

<a href="https://imgur.com/wMnZnnN"><img src="https://i.imgur.com//wMnZnnN.png" title="source: imgur.com" /></a>

#### 2.1 AiArenaHelper.sol

The `AiArenaHelper` contract is part of a system designed to generate and manage physical attributes for AI Arena fighters. These attributes play a crucial role in determining the characteristics and abilities of the fighters within the AI Arena ecosystem. The contract facilitates the creation of attributes based on DNA, generation, and other parameters, providing a foundation for the diversity and uniqueness of fighters within the system.

But this contract employs a modular approach by separating concerns into different functions and mappings. This allows for easier maintenance, upgrades, and enhancements to specific functionalities without affecting the overall system. Additionally, the contract utilizes Solidity's versioning pragma to ensure compatibility and security.

<a href="https://imgur.com/HjQMHsF"><img src="https://i.imgur.com//HjQMHsF.png" title="source: imgur.com" /></a>

#### 2.2 FigherFarm.sol

`FighterFarm` oversees AI Arena Fighter NFTs, handling creation, ownership transfer, and attribute updates. It operates under the ERC721 standard, with tailored functionalities for the AI Arena ecosystem.

The `FighterFarm` contract follows a modular architecture, separating different functionalities into distinct contracts such as `FighterOps`, `Verification`, `AAMintPass`, `AiArenaHelper`, and `Neuron`. This modular design allows for easier maintenance, upgradability, and scalability of the system. Each contract handles specific tasks, promoting code reusability and reducing complexity within the main contract.

<a href="https://imgur.com/XHdWMIS"><img src="https://i.imgur.com//XHdWMIS.png" title="source: imgur.com" /></a>

#### 2.3 FighterOps.sol

According to my understanding they provided Solidity contracts are part of the AI Arena ecosystem, specifically focusing on managing and creating non-fungible tokens (NFTs) representing fighters within the game. These contracts facilitate the creation, ownership, and interaction with these NFTs, enabling players to own and trade unique digital assets.

But the architecture follows a modular design, with multiple contracts fulfilling specific functionalities such as managing fighters (FighterFarm.sol), defining fighter attributes (FighterOps.sol), and handling mint passes (AAMintPass.sol). This modular approach enhances code readability, maintenance, and extensibility.

<a href="https://imgur.com/SbYpcJg"><img src="https://i.imgur.com//SbYpcJg.png" title="source: imgur.com" /></a>

#### 2.4 GameItems.sol

According to me the simplest explanation of this contract is that these contracts, integral to the AI Arena game by ArenaX Labs Inc., manage Fighters and Game Items, facilitating their creation, management, and trading. Their core function is to ensure secure and efficient user interactions within the game ecosystem.

According to my observation One architectural improvement in these contracts is the use of the ERC-1155 standard for managing both Fighters and Game Items. This standard allows for the efficient management of multiple token types within a single contract, reducing gas costs and simplifying interactions for users.

<a href="https://imgur.com/HdiPgrI"><img src="https://i.imgur.com//HdiPgrI.png" title="source: imgur.com" /></a>

#### 2.5  MergingPool.sol

The `MergingPool` contract facilitates the potential acquisition of a new fighter NFT by users. It operates as a rewards system where users can earn points, and winners, determined periodically, can claim rewards in the form of new fighter NFTs. This contract serves as a gamification mechanism within the AI Arena ecosystem, incentivizing user engagement and participation in ranked battles.

But the one fact is that the contract design adheres to the separation of concerns principle by utilizing separate contracts for distinct functionalities, such as the FighterFarm contract for managing fighter NFTs and the RankedBattle contract for conducting battles. This modular approach enhances maintainability, readability, and upgradability of the overall system.

<a href="https://imgur.com/jLHBYTv"><img src="https://i.imgur.com//jLHBYTv.png" title="source: imgur.com" /></a>

#### 2.6 Neuron.sol

These contracts serve as the foundation of the ArenaX Labs Inc. ecosystem, enabling the management and utilization of critical platform components. They ensure integrity through mechanisms for token handling, staking, and reward distribution, bolstered by access control features. Overall, they establish a robust and secure environment for users within the ArenaX ecosystem.

The contract architecture is modular and scalable, using OpenZeppelin libraries for standard functions like managing tokens and access control. This approach makes it easier to maintain and expand the system with new features. Additionally, the contracts follow smart contract best practices, using standard interfaces and security measures to reduce risks.

<a href="https://imgur.com/t6p6kpT"><img src="https://i.imgur.com//t6p6kpT.png" title="source: imgur.com" /></a>

#### 2.7 RankedBattle.sol

The RankedBattle contract, developed by ArenaX Labs Inc., facilitates staking NRN tokens on fighters, tracks battle records, calculates rewards based on outcomes and stakes, and allows claiming of rewards. It's a vital component of a broader ecosystem for managing battles and rewards in decentralized applications.

The contract adopts a modular architecture, interfacing with FighterFarm, VoltageManager, MergingPool, Neuron, and StakeAtRisk contracts. This approach enhances code organization, maintenance, and scalability, leveraging events and mappings for efficient data handling.

<a href="https://imgur.com/AQNVX07"><img src="https://i.imgur.com//AQNVX07.png" title="source: imgur.com" /></a>

#### 2.8 StakeAtRisk.sol

RankedBattle and StakeAtRisk are integral components of a decentralized application (dApp) created by ArenaX Labs Inc. These contracts enable users to participate in ranked battles by staking NRN tokens on their fighters and managing the associated risks. RankedBattle manages staking, battle records, reward calculations, and reward claiming, while StakeAtRisk focuses on mitigating the loss of NRN tokens during battles.

One architectural improvement could be the introduction of an Oracle system to fetch external data, such as fighter statistics or match outcomes, to enhance the accuracy and fairness of battles. Additionally, implementing a modular design pattern could improve code readability and maintainability, making it easier to upgrade and extend the protocol in the future.

<a href="https://imgur.com/eAjaCvS"><img src="https://i.imgur.com//eAjaCvS.png" title="source: imgur.com" /></a>

#### 2.9 VoltageManager.sol

The `VoltageManager` contract is a part of the AI Arena ecosystem developed by ArenaX Labs Inc. It manages the voltage system within the game, allowing users to replenish and spend voltage for various in-game actions.

The contract follows a modular architecture, separating concerns by utilizing separate contracts for different functionalities. For example, it imports the `GameItems`contract to handle interactions with in-game items. This modular approach enhances readability, maintainability, and allows for easier upgrades and modifications.

<a href="https://imgur.com/tGYihVo"><img src="https://i.imgur.com//tGYihVo.png" title="source: imgur.com" /></a>

## 3.  Main Features Of Protocol

I'll exert oneself to conclude main features of protocol contract by contract in a sequence.

#### 3.1 AiArenaHelper

AiArenaHelper is a utility contract designed to provide various auxiliary functions and services to users within the AI Arena ecosystem. It may include functionalities such as calculating battle outcomes, providing recommendations or insights to users, managing game events or tournaments, and offering support for community-driven initiatives or activities. This contract serves to enhance the overall user experience and engagement within the AI Arena platform by offering useful tools and services.
                       
#### 3.2 FighterFarm

The FighterFarm contract serves as the backbone for managing AI Arena Fighter NFTs. It enables the creation, ownership, and redemption of these NFTs within the AI Arena ecosystem. Key features include minting new NFTs, transferring ownership, and updating fighter attributes. The contract follows the ERC721 standard for NFT functionality and includes additional features specific to the AI Arena platform.

#### 3.3 FigherOps 

FighterOps is a contract dedicated to managing various operations related to fighters within the AI Arena ecosystem. It provides functionalities such as updating fighter attributes, merging fighters to create more powerful ones, and other operations that enhance the capabilities and characteristics of fighters. This contract plays a crucial role in maintaining the diversity and competitiveness of the fighter pool in the game.

#### 3.4 GameItems

The GameItems contract facilitates the creation, ownership, and trading of in-game items within the AI Arena ecosystem. It supports various actions such as minting new items, transferring ownership, and updating item attributes. This contract adheres to the ERC721 standard for NFTs and includes functionalities tailored to the needs of the AI Arena game.

#### 3.5 MergingPool

The MergingPool contract facilitates the merging of fighters to create new and more powerful entities. It manages the merging process by combining the attributes of multiple fighters to generate unique offspring with enhanced abilities. This contract incentivizes users to strategically merge their fighters to gain a competitive advantage in battles, adding an element of strategy and progression to the gameplay experience.

#### 3.6 Neuron

Neuron is an ERC20 token contract that serves as the native currency within the AI Arena ecosystem. It enables users to stake tokens on fighters, participate in battles, and earn rewards. The contract implements standard ERC20 functionalities such as token transfers and approvals, along with additional features specific to the AI Arena platform, such as staking mechanisms.

#### 3.7 RankedBattle

RankedBattle is a smart contract designed to manage ranked battles between fighters within the AI Arena game. It facilitates the staking of NRN tokens on fighters, tracks battle records, calculates rewards based on battle outcomes and staked amounts, and allows users to claim accumulated rewards. The contract plays a crucial role in ensuring fair and transparent battles while incentivizing user participation.

#### 3.8 StakeAtRisk

The StakeAtRisk contract addresses the risk of losing NRN tokens during battles within the AI Arena game. It tracks the amount of NRN tokens staked by users and ensures that they can reclaim their tokens if they win a battle. Additionally, the contract manages the distribution of stakes and rewards, providing a mechanism to mitigate potential losses for users.

#### 3.9 VoltageManager

VoltageManager is responsible for managing the voltage system in the AI Arena game. It tracks the voltage levels of users and replenishes their voltage periodically. The contract allows users to spend voltage for various in-game actions, such as participating in battles. It ensures a balanced gameplay experience by regulating the availability of voltage to users.

## 4. Contracts Call Graph Architecture

Here is the overview how each and every componenet is intracting with each other in protocol.

#### 4.1 AiArenaHelper.sol

This contract generates and manages the physical attributes of AI Arena fighters, determining their visual appearance and characteristics within the game ecosystem. It plays a crucial role in defining the unique traits and attributes of each fighter, enhancing the diversity and richness of the gameplay experience.

<a href="https://imgur.com/7a4kLnR"><img src="https://i.imgur.com//7a4kLnR.png" title="source: imgur.com" /></a>

#### 4.2 FighterFarm.sol

This contract oversees the lifecycle of AI Arena Fighter NFTs, facilitating their creation, ownership transfers, and redemption. It enables minting new NFTs either from a merging pool or via mint passes, ensuring efficient management of the NFT ecosystem within the game.

<a href="https://imgur.com/hnbnabT"><img src="https://i.imgur.com//hnbnabT.png" title="source: imgur.com" /></a>

#### 4.3 FigherOps.sol

This library defines the structure of a Fighter and provides functions for retrieving fighter data, contributing to streamlined access to fighter information within the AI Arena ecosystem.

<a href="https://imgur.com/fD69zyo"><img src="https://i.imgur.com//fD69zyo.png" title="source: imgur.com" /></a>

#### 4.4 GameItems.sol

This contract serves as a repository for various game items utilized within AI Arena, enabling their management and utilization throughout the gaming ecosystem.

<a href="https://imgur.com/0eiov3q"><img src="https://i.imgur.com//0eiov3q.png" title="source: imgur.com" /></a>

#### 4.5 MergingPool.sol

This contract facilitates the opportunity for users to acquire a new fighter NFT through various mechanisms such as merging existing fighters or redeeming special items.

<a href="https://imgur.com/xpEhvmI"><img src="https://i.imgur.com//xpEhvmI.png" title="source: imgur.com" /></a>

#### 4.6 Neuron.sol

The Neuron token serves multiple purposes within the platform, encompassing staking, governance participation, and reward distribution.

<a href="https://imgur.com/atwgIII"><img src="https://i.imgur.com//atwgIII.png" title="source: imgur.com" /></a>

#### 4.7 RankedBattle.sol

The RankedBattle contract facilitates staking NRN tokens on fighters, monitors battle records, computes and allocates rewards based on battle results and staked quantities, and enables the claiming of accrued rewards.

<a href="https://imgur.com/plcWe34"><img src="https://i.imgur.com//plcWe34.png" title="source: imgur.com" /></a>

#### 4.8 StakeAtRisk.sol

The StakeAtRisk contract enables the RankedBattle contract to oversee the staking of NRN tokens that are at risk during battles.

<a href="https://imgur.com/LpQRDDo"><img src="https://i.imgur.com//LpQRDDo.png" title="source: imgur.com" /></a>

#### 4.9 VoltageManager.sol

The VoltageManager contract facilitates the management of voltage for game items and includes functions for using and replenishing voltage.

<a href="https://imgur.com/Ng53Mo4"><img src="https://i.imgur.com//Ng53Mo4.png" title="source: imgur.com" /></a>

## 5. Security Improvement

When it comes to writing security improvements, it's important to have a clear plan in mind. Here i'll explain security improvements contract by contract in the sequence.

#### 5.1 AiArenaHelper.sol

1. The contract uses the `transferOwnership` function to ensure that ownership can only be transferred by the current owner, preventing unauthorized access to critical functions.
2. Access control modifiers are utilized to restrict certain functions to only be callable by the contract owner, reducing the attack surface by limiting privileged actions to trusted entities.
3. Careful consideration is given to input validation and boundary checks to prevent potential vulnerabilities such as overflow or underflow.

#### 5.2 FighterFarm.sol

The contract implements access control modifiers to restrict sensitive functions to authorized addresses only. Critical functions such as ownership transfer, contract initialization, and staking status updates are protected by access control checks, ensuring that only authorized entities can execute them.

#### 5.3 FighterOps.sol

1. Role-Based Access Control: Certain functions are restricted to specific roles (e.g., owner, staker) to prevent unauthorized access.
2. Signature Verification: Critical functions like claiming fighters require signature verification to ensure authenticity.
3. Safe Transfer Functions: Utilization of safe transfer functions to prevent accidental loss of assets during transfers.

#### 5.4 GameItems.sol

One security improvement is the implementation of access control mechanisms throughout the contracts. Only authorized users, such as admins and contract owners, can perform critical actions such as adjusting transferability, minting new items, and setting token URIs. This helps prevent unauthorized access and ensures the integrity of the game ecosystem.

#### 5.5 MergingPool.sol

1. Access Control: Certain critical functions, such as picking winners and adjusting admin access, are restricted to authorized addresses, preventing unauthorized manipulation.
2. External Contract Interaction: Interaction with external contracts is limited to trusted contracts, reducing the attack surface and mitigating potential vulnerabilities.
3. State Variable Encapsulation: Sensible state variables are made private or internal, preventing direct external access and ensuring data integrity.

#### 5.6 Neuron.sol

1. Implementation of RBAC using OpenZeppelin's AccessControl contract to restrict unauthorized access to critical functions.
2. Proper validation checks and access modifiers to ensure that only authorized addresses can execute sensitive operations.
3. Utilization of standard ERC20 functionality for token management, reducing the risk of vulnerabilities and ensuring compatibility with existing infrastructure.

#### 5.7 RankedBattle.sol

1. Access Control: Only authorized addresses, such as owners and admins, can perform critical actions like adjusting admin access, setting game server addresses, and managing round transitions.
2. Permissioned Operations: Certain functions require specific conditions to be met, such as owning the fighter being staked or having sufficient NRN balance.
3. Event Logging: Important events, such as staking, unstaking, and claiming, are logged with associated data for transparency and auditability.

#### 5.8 StakeAtRisk.sol

Implementing access control modifiers and permission checks in critical functions to ensure that only authorized contracts or addresses can perform sensitive operations. Additionally, conducting thorough security audits and implementing best practices, such as using SafeMath for arithmetic operations and minimizing the attack surface, can enhance the security of the protocol.

#### 5.9 VoltageManager.sol

1. Access Control: Certain functions are restricted to specific addresses. For example, only the owner can transfer ownership and adjust admin access.
2. Permissioned Actions: Voltage spending is controlled based on permission settings, ensuring that only authorized addresses can spend voltage.
3. Input Validation: Functions include input validation to prevent unauthorized or invalid actions, enhancing security.
   
## 6. Systematic Risk 

If ownable or approveable address get compromise whole protocol will break and become useless. Try using multisig for admins addresses. 

#### 6.1 AiArenaHelper.sol

1. The reliance on the owner for certain critical functions like transferring ownership could introduce a systematic risk if the owner's account is compromised or misused.
2. The contract's dependence on external data sources or contracts for attribute probabilities may introduce systematic risks related to data availability and reliability.

#### 6.2 FighterFarm.sol

The `FighterFarm` contract minimizes systematic risk by enforcing constraints on critical operations such as minting, transferring, and updating fighter attributes. Access control mechanisms prevent unauthorized access to sensitive functions, reducing the risk of exploits or unauthorized actions that could compromise the integrity of the system.

#### 6.3 FighterOps.sol

1. Smart Contract Bugs: Potential vulnerabilities in the smart contracts could lead to exploits or loss of funds.
2. External Dependency Risks: Reliance on external contracts and libraries introduces the risk of vulnerabilities or changes in behavior.

#### 6.4 GameItems.sol 

One systematic risk is potential vulnerabilities in the smart contracts that could be exploited by malicious actors. These vulnerabilities could lead to financial losses or disruption of the game ecosystem. Regular security audits and thorough testing are essential to mitigate these risks and ensure the robustness of the contracts.

#### 6.5 MergingPool.sol

The primary systematic risk lies in the centralization of admin privileges. If compromised, admins could unfairly influence winner selection or disrupt the reward distribution process.

#### 6.6 Neuron.sol

1. Inherent risks associated with smart contract development, including bugs, vulnerabilities, and unforeseen interactions with external contracts.
2. Potential for misuse or exploitation of privileged roles, leading to unauthorized minting, burning, or allocation of tokens.
3. Dependency on external libraries and infrastructure, which may introduce additional risk factors such as versioning issues or security vulnerabilities.

#### 6.7 RankedBattle.sol

1. Diversification: By allowing users to stake on multiple fighters, the risk associated with a single fighter's performance is spread out.
2. Dynamic Reward Distribution: Rewards are distributed periodically based on battle outcomes and staked amounts, reducing the impact of individual events on overall rewards.
3. Continuous Monitoring: The contract tracks battle records and stake-at-risk amounts, enabling proactive risk management strategies.

#### 6.8 StakeAtRisk.sol

One systematic risk is the potential for smart contract vulnerabilities or bugs that could result in financial losses or unintended behavior. Additionally, reliance on external data sources or oracles introduces the risk of data manipulation or inaccuracies affecting battle outcomes and rewards distribution.

#### 6.9 VoltageManager.sol

The contract doesn't expose any systematic risks directly. However, improper access control or vulnerabilities in the imported contracts (like `GameItems`) could pose risks if not carefully audited.

## 7. Centralization Risk

Centralization risks in these contracts

#### 7.1 AiArenaHelper.sol

1. The centralized control of certain functions by the contract owner may introduce centralization risks, particularly if the owner's decisions significantly impact the functioning of the AI Arena ecosystem.
2. The ability to add or delete attribute probabilities for generations is centralized, potentially leading to disparities in attribute distribution if not managed transparently and fairly.

#### 7.2 FighterFarm.sol

The contract mitigates centralization risk by decentralizing ownership and control over fighter NFTs. Ownership of NFTs is managed on-chain through the ERC721 standard, allowing users to have full ownership and control over their assets without relying on centralized authorities. Additionally, access control mechanisms prevent any single entity from monopolizing control over critical contract functions.

#### 7.3 FighterOps.sol

1. Owner Privileges: The contract owner has significant control over contract functions, potentially leading to centralization of power.
2. Staker Role: Staker role assignment may centralize the ability to stake fighters, depending on how it's managed.

#### 7.4 GameItems.sol

There is a centralization risk associated with the ownership and control of the contracts by a single entity or group of entities. If these entities act maliciously or become compromised, they could manipulate the game ecosystem or exploit users. Implementing decentralized governance mechanisms and ensuring transparency in contract ownership can help mitigate this risk.

#### 7.5 MergingPool.sol

The contract's reliance on admin-controlled functions for winner selection and access control introduces centralization risks. Admins hold significant power over the protocol's operation, posing potential vulnerabilities if their privileges are misused or compromised.

#### 7.6 Neuron.sol

1. Centralization of certain administrative functions, such as role assignment and contract ownership, may pose risks of single points of failure or manipulation.
2. Dependency on trusted entities for contract deployment and maintenance, which could introduce centralization risks if not adequately decentralized or distributed.

#### 7.7 RankedBattle.sol

To mitigate centralization risk, the contract implements:

1. Decentralized Governance: Admin controls are limited to a small set of addresses, and mechanisms are in place to prevent any single entity from having undue influence over the protocol.
2. Distributed Staking: Staked NRN tokens are distributed across multiple fighters and users, reducing the concentration of power in the hands of a few.

#### 7.8 StakeAtRisk.sol

The protocol's reliance on a central treasury address for fund management and the presence of privileged contract addresses, such as the `RankedBattle` contract, introduces centralization risks. If these central entities are compromised or mismanaged, it could undermine the integrity and fairness of the protocol.

#### 7.9 VoltageManager.sol

There's a centralization risk associated with the owner address, which has significant control over the contract, including ownership transfer and admin access adjustments. Over-reliance on a single entity or address for such critical functions may introduce centralization risks.

## 8. Approach Evaluating CodeBase

Day 1

- I started this contest 2 days ago, i then went to C4 homepage and read its overview and then i check the Scope which contracts are given.
- I Cloned it to GitHub, then I set it up and finally I started to understand the code base. 
- I had to test it, for testing i executed foundry tests then the tests were executed and the one thing i see in testing there is lacking of invariant or fuzz testing. I would suggest the team to include fuzz testing and invariant testing. 

Day 2

- Made notes to understand the code base structure.
- For making diagram i used miro, miro helps me to understand the codebase in a better way.
- wrote Analysis Report.

## 9. Time Spent

- 14 Hours of focused work.

### Time spent:
14 hours