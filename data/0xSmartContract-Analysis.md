# 🛠️ Analysis - AI Arena Audit 
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |Other recommendations | What is unique? How are the existing patterns used? |
|h) |New insights and learning from this audit | Things learned from the project |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2024-02-ai-arena

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2024-02-ai-arena?tab=readme-ov-file#tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [AI Arena](https://docs.aiarena.io/gaming-competition) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/code-423n4/2024-02-ai-arena/blob/main/slither.txt)| |
|5|Test Suits|[Tests](https://github.com/code-423n4/2024-02-ai-arena?tab=readme-ov-file#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2024-02-ai-arena?tab=readme-ov-file#scoping-details)|
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2024-02-ai-arena?tab=readme-ov-file#scope)||

## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit

In addition, the auditor can decide on the question of where should I end the audit by examining these metrics;

Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

<img width="730" alt="image" src="https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/37ea9504-fdca-40cd-a4f0-aa4ead9ec6d6">




</br>
</br>
</br>
</br>

## Project - Entry Point - FighterFarm.sol ;

1. The `reRoll` function implements business logic after the `_neuronInstance.transferFrom` call. This may result in the `reRoll` function being called again and causing unexpected behavior. Always implement business logic before the call

2. **DoS**: The `redeemMintPass` function uses a loop to burn multiple mint passes. If the `burn` operation for a `mintpassIdToBurn` fails, the entire operation is rolled back. This could lead to a malicious user making this function of the contract unusable, causing the transaction to fail.
    - **Recommendation**: Manage each `burn` operation separately using `try-catch` and log the failed ones.

3. **Unchecked Return Values**: The return value of the `_neuronInstance.transferFrom` call in the `reRoll` function is not checked. This could cause the transaction to fail silently if the transfer fails.
    - **Recommendation**: Check the return value to verify that the transfer was successful.

4. **Front-Running**: `claimFighters` and `redeemMintPass` functions allow users to create NFTs based on certain parameters. Publicly publishing these parameters (e.g., `modelHashes`) on the blockchain carries the risk that malicious users can see the transaction and take action in their favor (e.g., pre-snatch more valuable NFTs).
    - **Recommendation**: Prevent such front-running attacks by using commit-reveal scheme.

### Gas Optimizations

1. **Gas Optimization**: Frequently used `length` calls within `for` loops can increase gas costs. Especially in the `redeemMintPass` and `claimFighters` functions, the loop calls `fighters.length` at each iteration.
    - **Suggestion**: Store `length` in a variable before the loop and use this variable inside the loop.

2. **Use of ERC721 and ERC721Enumerable**: `ERC721Enumerable` maintains a list of all tokens, which can significantly increase gas costs for large collections.
    - **Recommendation**: If your collection is large and the list of tokens owned by each user is not needed frequently, you may consider removing `ERC721Enumerable`.

![image](https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/000edf8a-2cb9-4cd2-81b1-9d64985f916b)

##### Note: Please click on the image to enlarge it:



</br>
</br>
</br>
</br>

## RankedBattle.sol

#### 1. DoS (Denial of Service) Risk:
- The `updateBattleRecord` function may consume a lot of gas under certain conditions or be exploited by a malicious user. For example, very high `eloFactor` values may cause the operation to fail.

#### 2. Magic Numbers:
- Constant values such as `VOLTAGE_COST` or `bpsLostPerLoss` are used directly in various places in the code. Documenting the meaning of these values and why they were chosen makes the code easier to read and maintain.

#### 2. Function Visibility and Modifiers:
- Using `modifier` for authorization checks reduces code repetition and increases readability.

### Gas Optimizations

#### 1. Optimization on Frequently Called Functions:
- It is important to optimize calculations to reduce gas costs in frequently called functions such as `updateBattleRecord`. For example, calculations with constant values can be precomputed.

#### 2. Loops and Big Data Structures:
- Using loops on large data structures can increase gas cost. In functions like `getUnclaimedNRN`, limiting the maximum number of rounds users can take or organizing data structures more efficiently can save gas.

#### 3. Optimization of Access to State Variables:
- Gas can be saved by copying frequently accessed `state` variables to `memory` variables and performing operations on these variables.


<img width="699" alt="image" src="https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/29c4a0a4-3a1b-4949-9424-f910ab914ba3">

##### Note: Please click on the image to enlarge it:

</br>
</br>
</br>
</br>

## MergingPool.sol

This Solidity contract includes a management and reward distribution mechanism that provides a number of special functionalities. Below are brief summaries of the contract's functions and constructor:

### Constructor
- **Constructor**: The contract owner sets the ranked battle contract and warrior farm contract addresses. The contract identifies the owner as an admin.

### External Functions
- **transferOwnership**: Transfers contract ownership to another address. Can only be summoned by the current owner.
- **adjustAdminAccess**: Grants or removes admin access to a specific address. Can only be summoned by its owner.
- **updateWinnersPerPeriod**: Updates the number of winners per contest period. It can only be called by admins.
- **pickWinner**: Picks the winners for the current round. The number of winners must match the expected number of winners per period. It can only be called by admins.
- **claimRewards**: Allows users to collectively claim rewards for multiple rounds. The user can only claim rewards once per round.

### Public Functions
- **addPoints**: Adds merge pool points to a warrior. It can only be summoned by the ranked battle contract address.
- **getFighterPoints**: Gets points of fighters up to the specified maximum token ID.

### Architecture Summary
This contract provides a reward and management system for warriors. Warriors can earn points through ranked battles and other events. Admins can select winners during certain periods, and these winners can claim their rewards to create new fighters with specific model URIs, model types, and special features. Users can view and claim the number of rewards they have earned.

The contract protects management functions with a strict access control mechanism, allowing only certain addresses (owner and admins) to perform management functions and reward distribution. This ensures the security of the contract and the correct use of its functionality. It also offers a mechanism to manage points from ranked battles and other events, rewarding warriors' performance and participation.

![image](https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/d3feebb6-95b4-4338-8b7a-cde09668c284)

##### Note: Please click on the image to enlarge it:

Contract Details -  MergingPool.sol ; 

![image](https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/a395678e-9d63-4be1-a8c4-993d4ea9778c)

##### Note: Please click on the image to enlarge it:

</br>
</br>
</br>
</br>

##  AiArenaHelper.sol

It functions as the AI Arena utility and enables the creation of physical attributes for a particular fighter.

### State Variables

- `attributes`: List of attributes that fighters can have (e.g. "head", "eyes", etc.).
- `defaultAttributeDivisor`: Default values of each attribute's DNA dividers.
- `_ownerAddress`: Address of the contract owner.
- `attributeProbabilities`: A mapping that keeps track of attribute probabilities for each generation.
- `attributeToDnaDivisor`: A mapping that keeps track of the DNA dividers specific to each attribute.

### Constructor

- When starting the contract, sets the trait probabilities for generation 0 and initializes the DNA splitter of each trait with default values.

### External Functions

- `transferOwnership`: Transfers contract ownership to another address.
- `addAttributeDivisor`: Adds or updates DNA dividers for each attribute.
- `createPhysicalAttributes`: Creates a fighter's physical attributes based on the given DNA, lineage, icon type and dendroid status.

###Public Functions

- `addAttributeProbabilities`: Adds attribute probabilities for a given generation.
- `deleteAttributeProbabilities`: Deletes attribute probabilities for a given generation.
- `getAttributeProbabilities`: Returns attribute probabilities for a given generation and trait.
- `dnaToIndex`: Converts the given DNA and its rarity into a trait probability index.

### Architectural Summary

This contract is designed to manage the characteristics of fighters in the AI Arena. The contract contains complex logic used to create fighters' physical attributes. It governs DNA splitters for each trait and trait probabilities that vary across generations.


![image](https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/46f4be1e-ecfd-48c0-92a8-9b0d41af5ee4)

##### Note: Please click on the image to enlarge it:

1. **Hard Coded Limitations**: Within the `createPhysicalAttributes` function there are hard coded control structures for some attributes (e.g. special cases based on `iconsType` values). This approach may limit the extensibility and flexibility of the contract.
    - **Recommendation**: Consider managing such controls through an external configuration or management mechanism to enable more dynamic management of features and behaviors.



2. **Role Based Access Control**: The contract authorizes the owner to perform administrative functions (e.g. `transferOwnership`, `addAttributeDivisor`). However, this may not be enough for growing projects that may require more complex access control mechanisms.
    - **Recommendation**: Create a more flexible and secure role-based access control system using OpenZeppelin's `AccessControl` contract.

3. **Data Validation**: Validating input data is important, especially when it comes to outsourced parameters. Functions such as `addAttributeProbabilities` and `addAttributeDivisor` check input lengths, but more extensive validations may be required.
    - **Recommendation**: Add comprehensive validation mechanisms to ensure accuracy and validity of input data.


</br>
</br>
</br>
</br>

##  GameItems.sol

This contract; Provides management of in-game items for AI Arena. It supports multiple token types using the ERC1155 standard.



1. **DoS by Block Gas Limit**: The `mint` function, especially in combination with the `finiteSupply` control and the `dailyAllowance` control, performs multiple operations in a loop. If these transactions consume too much gas, the block may reach its gas limit and cause transactions to fail.
    - **Recommendation**: Limit the maximum amount users can mint at once or optimize transactions to reduce gas usage.

2. **Unchecked External Call Return Values**: The return value of the `_neuronInstance.transferFrom` call is not checked in the `mint` function. This could cause the transaction to fail silently if the transfer fails.
    - **Recommendation**: Check the return value of the ERC20 `transferFrom` function and throw an appropriate error message if the operation fails.


3. **Magic Strings and Numbers**: "Magic strings" and "magic numbers" are used in many places in the contract, especially in URI management and time intervals.
    - **Recommendation**: Make the code easier to read and maintain by replacing magic numbers and strings with constants.

4. **Role-Based Access Control (RBAC)**: Managing roles such as `isAdmin` and `allowedBurningAddresses` may require more complex access control mechanisms.
    - **Recommendation**: Provide more flexible and secure role-based access control using OpenZeppelin's `AccessControl` contract.
    - 

![image](https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/1d8d555c-3e84-4fe2-aba5-7746e2578a7d)

##### Note: Please click on the image to enlarge it:


</br>
</br>
</br>
</br>

## c) Test analysis
### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content.

Test Coverage and Depth
Wide Function Testing: Test files cover many parts of your contracts (%90 Coverage). This means you check if every part of your system works in different situations.

Both Good and Bad Scenarios: Test files look at both when things go right and when they should fail and give an error. This makes sure your contracts can handle unexpected situations well.

Testing Strategies and Approaches
Looking at Edge Cases: Test files think about special or extreme situations to make sure your contracts can handle them.
Focus on Security: Tests like `RankedBattleTest` check important parts of your contracts to keep them safe. This is very important for smart contracts.

Testing How Things Work Together: Test files  also see how different contracts work with each other. This shows that your whole project works well as one system.



### What could they have done better?





-  1) If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;


<img width="1522" alt="image" src="https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/01efd435-cc9e-4282-ad45-08667a62d7fe">


Ref: https://xin-xia.github.io/publication/icse194.pdf

</br>
</br>

-   2) Overall line coverage percentage provided by your tests : 90 (As a generally accepted safety rule, 100% test coverage is recommended)


</br>
</br>



-  3) Test Tools/Technologies analysis of the project;

<img width="982" alt="image" src="https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/ef2ef752-5dc1-4d21-bdbe-65b95bf0a279">



</br>
</br>
</br>
</br>

## d) Security Approach of the Project

### Successful current security understanding of the project;

1- They manage the 1nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes. 



### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. 
https://immunefi.com/

5- Emergency Action Plan
In a high-level security approach, there should be a crisis handbook like the one below and the strategic members of the project should be trained on this subject and drills should be carried out. Naturally, this information and road plan will not be available to the public.
https://docs.google.com/document/u/0/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/mobilebasic#h.27dmpkyp2k1z

6- ChainAnalysis oracle
With the ChainAnalysis oracle, OFAC interaction can be blocked so that legal issues do not arise

7- We also recommend that you have an "Economic Audit" for projects based on such complex mathematics and economic models. An example Economic Audit is provided in the link below;
Economic Audit with [Three Sigma](https://panoptic.xyz/blog/panoptic-three-sigma-partnership)

8 - As the project team, you can consider applying the multi-stage audit model.

<img width="498" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/7ad49e46-4998-4bf2-830e-711039ea97a8">

Read more about the MPA model;
https://mpa.solodit.xyz/

9 - I recommend having a masterplan applied to project team members (This information is not included in the documents).
All authorizations, including NPM passwords and authorizations, should be reserved only for current employees. I also recommend that a definitive security constitution project be found for employees to protect these passwords with rules such as 2FA. The LEDGER hack, which has made a big impact recently, is the best example in this regard;

https://twitter.com/Ledger/status/1735326240658100414?t=UAuzoir9uliXplerqP-Ing&s=19


10 -Another security approach I can recommend is 0xDjango's security approaches in a project. This approach divides security into layers (Test, Automatic Tool, Individual Audit, Corporate Audit, Economic Audit) and aims to have each part done separately by experts in the field. I can also recommend this approach to you;

https://x.com/0xDjangoOnChain/status/1748402771684909094?s=20

</br>
</br>

## e) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2024-02-ai-arena/blob/main/bot-report.md

The Automated analysis identifies several security-related issues, such as potential Denial of Service (DOS) attacks through unbounded loops ([`M-01`], [`L-05`]), lack of proper validation for array lengths ([`L-06`]), and the absence of checks for null addresses ([`L-07`], [`L-12`]). Addressing these issues is crucial for preventing attacks that could compromise contract integrity and user assets.


</br>
</br>
</br>
</br>

## Centralization Risk

#### Owner Role:

1. **Single Point Failure Risk:** If the private key under the control of `_ownerAddress` is compromised or lost, the entire system is compromised.
2. **Risk of Abuse of Authority:** The contract owner may abuse his powers, for example, arbitrarily transfer funds or change roles in the system.


#### Code Pattern Used
By using `require(msg.sender == _ownerAddress);` it follows a pattern that ensures that certain functions can only be called by the contract owner. This indicates a centralized management model.

#### Suggestions

1. **Multisig Mechanism:** A multi-signature mechanism can be used that shares the powers of the contract owner among more than one address. This reduces the risk of a single point of failure and prevents abuse of authority.

2. **Transition to DAO Structure:** Moving project management and decision-making processes to a DAO (Decentralized Autonomous Organization) structure gives community members more say and reduces centralization.

3. **Time Lock and Escalation Mechanisms:** A time lock mechanism can be implemented so that critical functions can be called. This prevents sudden changes and gives the community a chance to review upgrades and changes.



</br>
</br>
</br>
</br>


##  f) Packages and Dependencies Analysis 📦

| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts)         | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts)                         | A lot of OZ contracts |-  Version `4.7.3` is used by the project, it is recommended to use the newest version `5.0.1`| 


</br>
</br>



</br>
</br>




## g) Other recommendations

✅ The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns; https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

✅ A good model can be used to systematically assess the risk of the project, for example this modeling is recommended; https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

✅ All staff accounts for the project should have control policies that require 2FA and must use 2FA wherever possible.
100% comprehensive security cannot be achieved based on smart contract codes alone.
Implement a more comprehensive policy to enforce non-SMS 2FA.
You can find the latest Simswap attack on Code4rena and details about it in this article: 
https://medium.com/code4rena/code4rena-twitter-x-incident-8b7f308a555d


✅ I recommend you to set up a `system.sol` basic architecture where all contracts are registered.
The entire system can revolve around a single contract, like SystemRegistry. This is the contract that ties all the other contracts together, and from this contract we should be able to list all the other contracts in the system. It's almost like a registry. 



✅ Analyze the gas usage of each function under various conditions in tests to ensure efficiency and identify potential optimizations.	



### Time spent:
24 hours