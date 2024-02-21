# Analysis - AI Arena Contest

![AI-Arena-Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FXEnWzzABSLk.0&w=96&q=75)

## Description overview of The AI Arena Contest

![Game](https://images.spr.so/cdn-cgi/imagedelivery/j42No7y-dcokJuNgXeA0ig/9e8dabe7-2ed2-49b2-ac71-63fa14c4ce9d/fighters/w=1920,quality=80)

**AI Arena** is a PvP platform fighting game where the fighters are AIs that were trained by humans. In the web3 version of the game, these fighters are tokenized via the **FighterFarm.sol** smart contract.

**Key Features and Concepts:**

- **Tokenized Fighters:** Each fighter NFT contains attributes that determine its appearance, abilities, and battle performance.
- **ERC20 Token ($NRN):** The game's native token, used for rewards and in-game purchases.
- **Ranked Battles:** Players can enter their NFT fighters into ranked battles to earn $NRN rewards.
- **Staking:** Players can stake their $NRN tokens to increase their rewards, but may lose part of their stake if they perform poorly.
- **Voltage:** Players have a limited amount of voltage per day, which is consumed during ranked battles. Voltage can be replenished naturally or purchased using batteries.
- **Off-Chain Game Logic:** The core game logic runs off-chain via servers, with the game server acting as an oracle for ranked match results.

**Mission and Goals:**

- **Democratize AI:** Make AI more accessible and understandable through gaming.
- **Monetization for AI Talent:** Provide direct monetization opportunities for AI researchers and players based on skill and merit.
- **Showcase Global AI Talent:** Highlight the individuals pushing the boundaries of AI development.

**Gameplay:**

- Players train their NFT characters through Imitation Learning, where the AI learns to play the game by copying the player's actions.
- Trained fighters are submitted into the arena for Ranked Battles, where they fight autonomously against opponents of similar skill levels.
- The objective is to train the most powerful AI NFT, climb the global leaderboard, and earn $NRN rewards.

**Unique Features:**

- **Human-in-the-Loop AI Training:** Players have direct involvement in training their AI fighters, providing a unique blend of human intuition and AI capabilities.
- **User-Friendly Interface:** The game's interface simplifies AI training and analysis, making it accessible to both casual and advanced users.

Overall, AI Arena aims to foster AI literacy and intuition through a fun and competitive gaming experience, while also showcasing the potential of Web3 for talent monetization and AI democratization.

## System Overview

### High-level System Overview

![AI-arena](https://private-user-images.githubusercontent.com/70327041/306673874-8633b72a-ead5-4e21-802c-41ba50d1cb68.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MDg1Mjg3NzQsIm5iZiI6MTcwODUyODQ3NCwicGF0aCI6Ii83MDMyNzA0MS8zMDY2NzM4NzQtODYzM2I3MmEtZWFkNS00ZTIxLTgwMmMtNDFiYTUwZDFjYjY4LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDAyMjElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwMjIxVDE1MTQzNFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTY0MjU2MWFkZTRhYmJkMzgzZTBiNWI5NDE2NjJjNjM5OGU4OWZiYzU4YjY4OGRlNjllZTc0NmZiYzdlYmQzZGMmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.93uyikn_ytNbrkoGEEjxT6yBMOhfcIzZkYQk9lkMDVQ)

### Scope

- **FighterFarm.sol**: This contract manages the creation, ownership, and redemption of AI Arena Fighter NFTs, allowing for minting new NFTs from a merging pool or through the redemption of mint passes which represent AI-controlled fighters in the AI Arena game. with functionalities such as updating fighter attributes, and managing staking statuses.

  - **Key Features:**

    - **Fighter Creation:** New fighters can be minted from a merging pool or through the redemption of mint passes.
    - **Fighter Attributes:** Fighters have various attributes, including physical appearance, element, weight, and machine learning model information.
    - **Fighter Staking:** Fighters can be staked to earn rewards, but cannot be traded while staked.
    - **Fighter Transfer:** Fighters can be transferred between addresses, but there are restrictions to prevent users from owning too many fighters or trading staked fighters.
    - **Rerolling:** Fighters can be rerolled to generate new random traits, at a cost of $NRN (the game's native token).
    - **Token URI:** Each fighter has a unique token URI that points to its metadata, including its appearance and other information.

  - **Additional Functionality:**

    - **Fighter Information:** Provides a way to retrieve all information related to a specific fighter.
    - **Contract URI:** Returns the URI where the contract metadata is stored.
    - **Token URI:** Returns the URI where the token metadata for a specific fighter is stored.
    - **Support for Interfaces:** Supports the ERC721 and ERC721Enumerable interfaces.

- **RankedBattle.sol**: This contract manages staking NRN tokens on fighters, tracks battle records, calculates and distributes rewards based on battle outcomes and staked amounts, and enables claiming of accumulated rewards.

  - **Key Features:**

    - Stake NRN tokens on fighters to earn rewards.
    - Track battle records and calculate rewards based on battle outcomes and staked amounts.
    - Claim accumulated rewards.
    - Adjust admin access.
    - Set the game server address.
    - Set the Stake at Risk contract address.
    - Instantiate the neuron contract.
    - Instantiate the merging pool contract.
    - Set the ranked NRN distribution amount for the current round.
    - Set the basis points lost per ranked match lost while in a point deficit.
    - Set a new round, making claiming available for that round.

- **AiArenaHelper.sol**: The `AiArenaHelper` contract generates and manages AI Arena fighters' physical attributes based on DNA, generation, and whether they are dendroids or not.

  - **Key Features:**

    - **DNA to attribute conversion:** The contract provides a function to convert a fighter's DNA and rarity rank into an attribute probability index. This index is used to determine the fighter's physical attributes.

    - **Custom icons:** The contract supports custom icons for certain attributes, such as the head, eyes, and hands. This allows for more customization and variety in fighter appearances.

    - **Dendroid support:** The contract can generate physical attributes for dendroids, which are a special type of fighter in the AI Arena game. Dendroids have unique physical attributes that are different from regular fighters.

- **FighterOps.sol**: The FighterOps library provides functions for creating and managing Fighter NFTs in the AI Arena game, including emitting events, extracting fighter attributes, and viewing fighter information.

- **GameItems.sol**:This contract represents a collection of game items used in the AI Arena game, allowing users to purchase, burn, and view information about these items.

- **MergingPool.sol**: The contract, `MergingPool`, is designed for a game where players can earn new fighter NFTs by participating in a merging pool. It allows users to accumulate points by battling with their fighters in a separate ranked battle contract. At the end of each round, a specified number of winners are selected from the top-scoring fighters, and these winners can claim new fighter NFTs.

- **Neuron.sol**: This contract, "Neuron," is an ERC20 token contract representing Neuron (NRN) tokens for AI Arena's in-game, featuring roles such as minters, spenders, and stakers, allowing for token minting, burning, and allowances setup, with an initial supply distribution to the treasury and contributors.

- **StakeAtRisk.sol**: This contract manages the NRNs (Neuron tokens) that are at risk of being lost during a round of competition in a `RankedBattle`.

- **VoltageManager.sol**: Manages the voltage system for AI Arena, allowing users to replenish and spend voltage.

### Roles

1. **FighterFarm.sol**:

   - **Owner Role:**

     - **Responsibilities:** The owner of the contract has certain privileged functions that they can execute, including:
       - Transferring ownership of the contract.
       - Increasing the generation of fighters.
       - Adding new addresses with staker roles.
       - Instantiating helper contracts and setting their addresses.
       - Setting the merging pool contract address.
     - **Control:** The owner role controls critical aspects of the contract's functionality and configuration, such as ownership, contract addresses, and certain parameters like generation and staker roles.
     - **Address:** Initially set during contract deployment, the owner address is the primary account that can execute these privileged functions.

   - **Staker Role:**

     - **Responsibilities:** Addresses with the staker role are allowed to update the staking status of fighters.
     - **Control:** Stakers have a specific role in managing the staking status of fighters, which can affect their tradability and other aspects of their behavior within the contract.
     - **Addition:** The contract owner has the authority to add new addresses to the staker role, effectively granting them permission to update fighter staking status.

   - **Delegated Address:** The address responsible for setting token URIs and signing fighter claim messages.

2. **RankedBattle.sol**:

   - **Owner**: The address that deploys the contract and has special privileges, including:

     - Transfer ownership of the contract.
     - Adjust admin access for other addresses.
     - Set the game server address.
     - Set the stake at risk contract address.
     - Instantiate contract instances (Neuron, FighterFarm, VoltageManager, MergingPool).

   - **Admin**: Addresses designated by the owner with admin access, capable of:

     - Adjusting admin access for other addresses.

   - **Game Server**: The address responsible for updating battle records. This role is crucial for maintaining the integrity and accuracy of battle records.

   - **Fighter Owner**: Addresses that own fighters and can stake or unstake NRN tokens for battles. They have control over their own fighters and stakes.

   - **Initiator**: Refers to the fighter owner who initiates a battle and expends voltage for the match. They are responsible for spending the required voltage.

3. **AiArenaHelper.sol**:

   - **Owner:** The owner address has the following roles:

     - Transfer ownership
     - Add attribute divisors
     - Add or delete attribute probabilities

4. **GameItems.sol**:

   - **Owner:** The owner address has the following roles:

     - Transfer ownership
     - Adjust admin access
     - Adjust transferability of game items
     - Set the Neuron contract address

   - **Admin:** Admins have the following roles:

     - Set allowed burning addresses
     - Set token URIs for game items
     - Create new game items

5. **MergingPool.sol**:

   - **Owner:** The address that has owner privileges and can transfer ownership, adjust admin access and configure parameters such as the number of winners per period..

   - **Admins:** Addresses that have been granted admin access by the owner.

   - **Ranked Battle Contract:** The address of the contract that manages ranked battles and awards merging pool points.

6. **Neuron.sol**:

   - **Owner:** The address that has owner privileges (initially the contract deployer).

   - **Minter:** Addresses that have been granted the minter role by the owner.

   - **Spender:** Addresses that have been granted the spender role by the owner.

   - **Staker:** Addresses that have been granted the staker role by the owner.

7. **VoltageManager.sol**:

   - **Owner:**
     - Can transfer ownership.
     - Can adjust admin access.
   - **Admin:**
     - Can adjust whether a given address is allowed to spend voltage.

### Chains supported

AI Arena is built on the Ethereum blockchain currently deployed on Arbitrum.

### Invariants Generated

- **FighterFarm.sol**:

  1. **Maximum Number of Fighters per Address**:

     - _Invariant_: `balanceOf(to) < MAX_FIGHTERS_ALLOWED`
     - _Explanation_: Ensures that no address can own more fighters than the maximum allowed limit (`MAX_FIGHTERS_ALLOWED`).

  2. **Ownership Verification for Token Transfer**:

     - _Invariant_: `_isApprovedOrOwner(msg.sender, tokenId)`
     - _Explanation_: Verifies that the sender is the owner of the fighter token or has been approved for transfer.

  3. **Fighter Staking Status Check**:

     - _Invariant_: `!fighterStaked[tokenId]`
     - _Explanation_: Ensures that the fighter being transferred is not currently staked and thus can be freely traded.

  4. **Maximum Rerolls Allowed Check**:

     - _Invariant_: `numRerolls[tokenId] < maxRerollsAllowed[fighterType]`
     - _Explanation_: Verifies that the number of rerolls performed on a fighter does not exceed the maximum allowed limit based on its fighter type.

  5. **Neuron Balance Verification for Reroll**:

     - _Invariant_: `_neuronInstance.balanceOf(msg.sender) >= rerollCost`
     - _Explanation_: Ensures that the sender has a sufficient balance of Neuron tokens (`NRN`) to cover the cost of rerolling a fighter.

  6. **Ownership Verification for Mint Pass Redemption**:

     - _Invariant_: `msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i])`
     - _Explanation_: Verifies that the sender owns the mint pass being burned for fighter minting.

  7. **Approval for Mint Pass Transfer**:

     - _Invariant_: `_mintpassInstance.approveSpender(msg.sender, rerollCost)`
     - _Explanation_: Approves the spender (in this case, the contract) to spend Neuron tokens on behalf of the sender for the rerolling process.

- **RankedBattle.sol**:

  1. **Staked Amount Check:**

     - _Invariant:_ `amountStaked[tokenId] >= 0`
     - _Explanation:_ Ensures that the staked amount of NRN tokens for a specific fighter token is always non-negative.

  2. **Total Staked Amount Calculation:**

     - _Invariant:_ `globalStakedAmount == Sum(amountStaked[tokenId])`
     - _Explanation:_ Verifies that the total staked amount of NRN tokens equals the sum of all individual stakings.

  3. **Staking Factor Calculation:**

     - _Invariant:_ `stakingFactor[tokenId] == sqrt((amountStaked[tokenId] + stakeAtRisk) / 10**18) || stakingFactor[tokenId] == 1`
     - _Explanation:_ Ensures that the staking factor for a token is correctly calculated based on the staked amount and stake at risk. If the staking factor is zero, it defaults to 1 to prevent division by zero errors.

  4. **Admin Access Control:**

     - _Invariant:_ `isAdmin[ownerAddress] == true`
     - _Explanation:_ Verifies that the owner address has admin privileges as intended.

  5. **Battle Result Update:**

     - _Invariant:_ `fighterBattleRecord[tokenId].wins + fighterBattleRecord[tokenId].ties + fighterBattleRecord[tokenId].loses == totalBattles`
     - _Explanation:_ Ensures that the total number of battles recorded for a fighter token equals the sum of wins, ties, and losses.

  6. **Claimable NRN Calculation:**

     - _Invariant:_ `amountClaimed[claimer] == Sum(accumulatedPointsPerAddress[claimer][roundId] * rankedNrnDistribution[roundId] / totalAccumulatedPoints[roundId])`
     - _Explanation:_ Verifies that the amount of unclaimed NRN tokens for a specific address is correctly calculated based on accumulated points and distribution.

  7. **Ownership Verification:**

     - _Invariant:_ `msg.sender == _ownerAddress || isAdmin[msg.sender] == true`
     - _Explanation:_ Ensures that only the owner address or admin addresses have permission to execute certain administrative functions.

- **AiArenaHelper.sol**:

  1. **Owner Privileges Verification:**

     - _Invariant:_ `msg.sender == _ownerAddress`
     - _Explanation:_ Ensures that only the owner address has authorization to execute certain privileged functions within the contract.

  2. **Ownership Transfer Authorization:**

     - _Invariant:_ `msg.sender == _ownerAddress` (in the `transferOwnership` function)
     - _Explanation:_ Ensures that only the current owner address can initiate the ownership transfer process.

  3. **Attribute Divisor Addition Authorization:**

     - _Invariant:_ `msg.sender == _ownerAddress` (in the `addAttributeDivisor` function)
     - _Explanation:_ Verifies that only the owner address is allowed to add attribute divisors.

  4. **Physical Attribute Generation:**

     - _Invariant:_ `createPhysicalAttributes` function should correctly calculate the final attribute probability indexes based on the input DNA, generation, icons type, and dendroid status.
     - _Explanation:_ Ensures that the physical attributes generated for a fighter are calculated accurately based on the provided parameters and contract logic.

  5. **Attribute Probabilities Update:**

     - _Invariant:_ `msg.sender == _ownerAddress` (in the `addAttributeProbabilities` and `deleteAttributeProbabilities` functions)
     - _Explanation:_ Verifies that only the owner address has the authority to update attribute probabilities for different generations.

  6. **Attribute Probability Retrieval:**

     - _Invariant:_ `getAttributeProbabilities(generation, attribute)` should return the correct attribute probabilities for a given generation and attribute.
     - _Explanation:_ Ensures that the contract accurately retrieves attribute probabilities when queried.

  7. **DNA to Index Conversion:**

     - _Invariant:_ `dnaToIndex` function should correctly convert DNA and rarity rank into an attribute probability index for a given attribute and generation.
     - _Explanation:_ Verifies that the conversion logic in the `dnaToIndex` function operates as intended, providing accurate attribute probability indexes.

- **GameItems.sol**:

  1. **Game Item Minting Authorization:**

     - _Invariant:_ (tokenId < `_itemCount`) && (`_neuronInstance.balanceOf(msg.sender) >= price`) && ((allGameItemAttributes[tokenId].finiteSupply == false) || ((allGameItemAttributes[tokenId].finiteSupply == true) && (quantity <= allGameItemAttributes[tokenId].itemsRemaining))) && ((dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp) || (quantity <= allowanceRemaining[msg.sender][tokenId]))
     - _Explanation:_ Ensures that the conditions for minting game items are met, including availability, balance, daily allowance, and price.

  2. **Transferability Check for Safe Transfer Function:**

     - _Invariant:_ `allGameItemAttributes[tokenId].transferable`
     - _Explanation:_ Ensures that game items can only be transferred if they are marked as transferable.

  3. **Daily Allowance Replenishment:**

     - _Invariant:_ Replenishment of daily allowance occurs correctly after the replenish interval has passed.
     - _Explanation:_ Verifies that the daily allowance is correctly replenished for users who buy game items after the replenish interval has elapsed.

  4. **URI Retrieval Override:**

     - _Invariant:_ Custom token URIs are correctly retrieved when available.
     - _Explanation:_ Ensures that custom token URIs are prioritized over the default URI retrieval logic.

- **MergingPool.sol**:

  1. **Winners per Period Update Authorization:**

     - _Invariant:_ `isAdmin[msg.sender]`
     - _Explanation:_ Ensures that only admins can update the number of winners per competition period.

  2. **Winner Selection Authorization:**

     - _Invariant:_ `isAdmin[msg.sender]`
     - _Explanation:_ Verifies that only admins can pick winners for the current round.

  3. **Reward Claiming Limit Check:**

     - _Invariant:_ Claiming rewards for each round does not exceed the total number of winners per period.
     - _Explanation:_ Verifies that the number of rewards claimed by a user does not exceed the total number of winners per period.

  4. **Unclaimed Rewards Retrieval Authorization:**

     - _Invariant:_ None
     - _Explanation:_ No authorization check needed as this function is public. However, it correctly calculates the unclaimed rewards for a specific address.

  5. **Total Points Calculation:**

     - _Invariant:_ `totalPoints` correctly reflects the sum of points added to all fighters.
     - _Explanation:_ Verifies that the `totalPoints` variable accurately tracks the total points added to all fighters in the merging pool.

  6. **Fighter Points Retrieval:**

     - _Invariant:_ `getFighterPoints` function correctly retrieves the points for multiple fighters up to the specified maximum token ID.
     - _Explanation:_ Ensures that the `getFighterPoints` function returns an array of points corresponding to the fighters' token IDs within the specified range.

## Approach Taken-in Evaluating The AI Arena Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the AI Arena Protocol.

    I start with the following contracts, which play crucial roles in the AI Arena protocol:

    **Main Contracts I Looked At**

    I start with the following contracts, which play crucial roles in the AI Arena protocol:

                        FighterFarm.sol
                        RankedBattle.sol
                        AiArenaHelper.sol
                        FighterOps.sol
                        GameItems.sol
                        MergingPool.sol
                        Neuron.sol
                        StakeAtRisk.sol
                        VoltageManager.sol

    I started my analysis by examining the intricate structure and functionalities of the `AI Arena` protocol, focusing on key contracts that play critical roles in the game ecosystem:- `FighterFarm.sol` this contract oversees the creation, ownership, and redemption of AI Arena Fighter NFTs. It facilitates minting new NFTs from a merging pool or via mint passes, representing AI-controlled fighters. Additionally, it manages fighter attribute updates and staking statuses.`RankedBattle.sol` Central to the competitive aspect of the game, this contract manages NRN token staking on fighters, battle record tracking, reward calculation, and distribution based on battle outcomes and staked amounts. It enables reward claiming and plays a crucial role in competitive gameplay.`AiArenaHelper.sol`: Responsible for generating and managing the physical attributes of AI Arena fighters based on DNA, generation, and dendroid status. It ensures each fighter's unique visual appearance and characteristics.
    `FighterOps.sol` This library offers functions for creating and managing Fighter NFTs, including event emission, attribute extraction, and information viewing, serving as a helper contract for fighter-related operations.`GameItems.sol` this contract represents a collection of in-game items, allowing users to purchase, burn, and view information about these items, enhancing the gameplay experience.`MergingPool.sol` designed for players to earn new fighter NFTs by participating in a merging pool, allowing point accumulation through battles in a ranked battle contract. Winners claim new fighter NFTs at the round's end.`Neuron.sol` as the ERC20 token contract for Neuron (NRN) tokens, the in-game currency, it facilitates token minting, burning, and allowance setup, featuring roles such as minters, spenders, and stakers.`StakeAtRisk.sol` managing NRNs at risk of loss during a `RankedBattle` round, it ensures player engagement by adding risk and reward elements to the competitive aspect.

2.  **Documentation Review**:

    Then went to Review [This Docs](https://docs.aiarena.io/gaming-competition) for a more detailed and technical explanation of AI Arena protocol.

3.  **Compiling code and running provided tests**:

4.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

## Codebase Quality

Overall, I consider the quality of the AI Arena protocol codebase to be Good. The code appears to be mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The AI Arena Protocol contracts demonstrates good maintainability through modular structure, consistent naming, and efficient use of libraries. It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space.                                                                                                                                                                                                                                                                                              |
| **Code Comments**                        | During the audit of the AI Arena contracts codebase, I found that some areas lacked sufficient internal documentation to enable independent comprehension. While comments provided high-level context for certain constructs, lower-level logic flows and variables were not fully explained within the code itself. Following best practices like NatSpec could strengthen self-documentation. As an auditor without additional context, it was challenging to analyze code sections without external reference docs. To further understand implementation intent for those complex parts, referencing supplemental documentation was necessary.                                        |
| **Documentation**                        | The documentation of the AI Arena project is quite comprehensive and detailed, providing a solid overview of how AI Arena protocol is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as `diagrams`, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors. |
| **Testing**                              | The audit scope of the contracts to be audited is 90% this is Good but aim to 100%.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| **Code Structure and Formatting**        | The codebase contracts are well-structured and formatted. but some order of functions does not follow the Solidity Style Guide According to the Solidity Style Guide, functions should be grouped according to their visibility and ordered: constructor, receive, fallback, external, public, internal, private. Within a grouping, place the view and pure functions last.                                                                                                                                                                                                                                                                                                             |
| **Error**                                | Use custom errors, custom errors are available from solidity version 0.8.4. Custom errors are more easily processed in try-catch blocks, and are easier to re-use and maintain.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| **Imports**                              | In AI Arena protocol the contract's interface should be imported first, followed by each of the interfaces it uses, followed by all other files.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the AI Arena protocol. These risks encompass concentration risk in ArbitrageSearch, Proposals.sol risk and more, third-party dependency risk, and centralization risks arising.

Here's an analysis of potential systemic and centralization risks in the contracts:

### Systemic Risks:

1. **FighterFarm.sol**

   - **Centralization**: The FighterFarm contract relies heavily on a single address (`_ownerAddress`) for critical operations such as transferring ownership, adding stakers, and setting contract parameters. This centralization poses a risk if the owner account is compromised or the owner becomes unresponsive.

2. **RankedBattle.sol**

   - **Stake at Risk Mechanism Risks**: The mechanism for handling stake at risk (StakeAtRisk contract) involves transferring NRN tokens between different contracts based on battle outcomes. There is a risk of loss or theft of these tokens if the mechanism is not implemented correctly or if there are vulnerabilities in the contracts involved.

3. **AiArenaHelper.sol**

   - **Lack of transparency:** The contract does not provide any way for users to view the attribute probabilities or DNA divisors used to generate fighter attributes. This could make it difficult for users to understand how the contract works and to verify the fairness of the attribute generation process.

4. **GameItems.sol**

   - **Price Oracle Risk:** The contract calculates the price of game items based on the balance of the `Neuron` contract (`_neuronInstance.balanceOf(msg.sender)`). If the price oracle mechanism in the `Neuron` contract is compromised or inaccurate, it could lead to incorrect pricing of game items, affecting user transactions and potentially loss.

5. **MergingPool.sol**:

   - **Denial of Service (DoS):** There's a risk of DoS attacks if the contract becomes popular, as the `pickWinner` function does not have a gas limit on iterating over the `winners` array. If `winners` array becomes excessively large, it could exceed the block gas limit, causing the function to fail, thereby denying legitimate users from claiming rewards.

6. **Neuron.sol**:

   - **Treasury Vulnerability**: The contract allows for setup of airdrops from the treasury address to multiple recipient addresses. If there are vulnerabilities in the setupAirdrop function or if the treasury address is compromised, it could result in unauthorized transfer of tokens to unintended recipients, leading to potential economic losses and impacting the overall stability of the token economy.

7. **VoltageManager.sol**:

   - **Potential Loss of User Funds**: Users could lose their voltage if the `useVoltageBattery` function is called mistakenly or maliciously without the intended game item balance check, causing a permanent reduction in their voltage without proper validation.

   - **Unrestricted Access to Voltage Adjustment**: The contract allows the owner to adjust admin access and modify the list of allowed voltage spenders without any restrictions or checks, potentially leading to abuse of power or unauthorized changes that could harm the system's integrity.

### Centralization Risks:

1. **FighterFarm.sol**

   - **Single Point of Control**: The `_ownerAddress_` has excessive control over the contract, including the ability to: Transfer ownership,Add and remove stakers,Set contract parameters (e.g., fighter generation, reroll costs)

   - **Potential Abuse**: The owner could potentially abuse their privileges to manipulate the contract or benefit themselves at the expense of other participants.

   - **Reduced Accountability**: With a single owner, there is less accountability for decision-making and contract management.

2. **RankedBattle.sol**

   - **Admin access**: The owner address has the ability to grant and revoke admin access to other addresses. If the owner address is compromised or malicious, they could grant admin access to unauthorized individuals who could then manipulate the contract's functionality.

   - **Game server address**: The owner address can set the game server address, which is responsible for updating battle records. If the game server address is compromised or malicious, it could provide inaccurate battle records or manipulate results to favor certain players.

   - **Contract instantiation**: The owner address is responsible for instantiating the neuron contract, merging pool contract, and stake at risk contract. If the owner address fails to instantiate these contracts correctly or uses malicious contracts, it could lead to system failures or vulnerabilities.

3. **AiArenaHelper.sol**

   - **Owner privileges:** The owner address has the ability to transfer ownership, add attribute divisors, and add or delete attribute probabilities. This gives the owner a significant amount of control over the contract and the fighter attributes that are generated.

   - **Single point of failure:** If the owner address is compromised or lost, it could lead to the contract becoming unusable or controlled by a malicious actor.

4. **GameItems.sol**

   - **Control over game items:** The owner has the ability to modify game item attributes, including transferability and supply, which could impact the value and utility of these items for users.

   - **Potential for abuse:** The owner could use their privileges to grant themselves or their associates unfair advantages, such as creating new game items with exclusive benefits or adjusting the daily allowance for specific users.

   - **Token URI Management:** The contract allows only administrators to set token URIs (`setTokenURI`). This centralization of control over token metadata could lead to censorship or manipulation of game item information, undermining the principles of decentralization and transparency.

5. **MergingPool.sol**:

   - **Owner privileges:** The contract owner has the ability to transfer ownership and adjust admin access. This gives the owner significant control over the contract and could lead to centralization of power.

   - **Admin privileges:** Admins have the ability to select winners, adjust admin access, and change the number of winners per period. This gives admins a high level of influence over the contract's operation and could lead to centralization of decision-making.

6. **Neuron.sol**:

   - **Owner privileges:** The contract owner has the ability to transfer ownership and adjust admin access. This gives the owner significant control over the contract and could lead to centralization of power.

   - **Admin privileges:** Admins have the ability to add and remove minters, stakers, and spenders, and adjust admin access. This gives admins a high level of influence over the contract's operation and could lead to centralization of decision-making.

7. **StakeAtRisk.sol**:

   - **RankedBattle contract privileges:** The RankedBattle contract has significant control over the StakeAtRisk contract, including the ability to set new rounds, reclaim NRNs, and update at-risk records. This centralization of power could lead to abuse or misuse of the contract.

   - **Treasury address:** The treasury address is hardcoded in the contract, which means that it cannot be changed without deploying a new contract. This centralization of control over the treasury could lead to funds being misappropriated or used for unauthorized purposes.

8. **VoltageManager.sol**:

   - **Owner privileges:** The owner has the ability to transfer ownership, adjust admin access, and adjust allowed voltage spenders. This centralization of power could lead to a single individual having too much control over the contract.

   - **Admin privileges:** Admins have the ability to adjust allowed voltage spenders, which could lead to a small group of individuals having too much control over who is allowed to spend voltage.

   - **Hardcoded game items contract address:** The address of the game items contract is hardcoded in the constructor, which means that it cannot be changed without deploying a new contract. This centralization of control over the game items contract could lead to problems if the game items contract is compromised or if the game items contract team makes changes that are not compatible with the voltage manager contract.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the AI Arena protocol.**

## Suggestions

### What’s unique?

- **AI-controlled fighters:** AI Arena is unique in that it features AI-controlled fighters that battle each other in a competitive environment. This introduces an element of unpredictability and skill to the gameplay, as players must adapt their strategies to counter the AI's behavior.
- **Merging pool:** The merging pool mechanic allows players to earn new fighter NFTs by participating in a merging pool and accumulating points through battles. This provides an additional way for players to acquire new fighters and enhance their collection.
- **Voltage system:** The voltage system adds a layer of resource management to the gameplay. Players must carefully manage their voltage to ensure they have enough to replenish their fighters' energy and perform special abilities.

- **Fighter NFTs with Dynamic Attributes**: One unique aspect is the creation and management of AI Arena Fighter NFTs with dynamic attributes such as physical appearance, element, weight, and machine learning model information. This adds depth to the gameplay by allowing for a wide variety of unique fighters with distinct characteristics.

- **Competitive Staking Mechanism**: The RankedBattle contract introduces a competitive staking mechanism where players can stake NRN tokens on fighters to earn rewards based on battle outcomes and staked amounts. This incentivizes strategic gameplay and enhances the competitive aspect of the game.

### What ideas can be incorporated?

- **Player-created content:** Allowing players to create and share their own custom fighters and game items could foster a sense of community and ownership within the AI Arena ecosystem.

- **Cross-chain compatibility:** Integrating with other blockchain platforms could expand the reach of AI Arena and attract a wider player base.

- **Tournaments and competitions:** Hosting regular tournaments and competitions could add an extra layer of excitement and challenge to the gameplay, providing players with opportunities to showcase their skills and win rewards.

## Conclusion

In general, The AI Arena protocol is a well-designed and innovative blockchain-based gaming platform that introduces several unique features to the NFT gaming space. The protocol's focus on AI-controlled fighters, merging pool mechanics, and voltage system provides a compelling and engaging gameplay experience.The AI Arena protocol exhibits strengths comprehensive testing, and detailed documentation, we believe the team has done a good job regarding the code.the AI Arena protocol has the potential to become a leading player in the NFT gaming industry. Its unique gameplay mechanics, coupled with its robust security measures, make it an attractive platform for both players and developers.It is highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.


### Time spent:
35 hours