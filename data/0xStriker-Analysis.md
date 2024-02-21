# Analysis AIArena Contest

![AIArena Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FXEnWzzABSLk.0&w=96&q=75)


## Description Overview of the AiArena
AI Arena is a gaming platform that incorporates human with AI. Users go on to the AI Arena platform, acquire NFTs representing their AI models. The users then train the AI models with a process called imitation learning, which basically means that the AI model observes the user's movements and techniques and learns from it, the more the AI model is trained the better it becomes. These AI models are then submitted to ranked matches and compete in matches with other AI models trained by other users. The winner of the match is payed in $NRN which is the AI Arena native token, and this is the ultimate goal, train the best AI model, compete in ranked matches and win rewards.

**In summary AIArena is**:
 - A platform where users acquire NFTs representing their AI model.
 - The users train the AI models and submit them in battles and win rewards if the AI model wins.

**The key contracts of the AI Arena for this audit are**

- **FighterFarm.sol**: This smart contract administers the creation, possession, and exchange of AI Arena Fighter NFTs. It facilitates the minting of new NFTs either from a merging pool or via the redemption of mint passes, in addition to managing ownership rights. Auditing this smart contract ensures that the creation and possession of the NFTs are implemented as intended.
- **RankedBattle.sol**: This contract enables users to stake/unstake NRN tokens on/from fighters, monitor battle records, compute and dispense rewards according to battle results and staked quantities, and permits the claiming of accrued rewards. A thorough audit of this contract is needed to ensure the rewards distribution, staking and unstaking of the tokens is as it supposed to be. 
- **Neuron.sol**: The Neuron contract is an ERC20 token representing Neuron (NRN) tokens, used within the AI Arena's in-game economy. It includes roles for minting, spending, and staking tokens, and features an initial minting for the treasury and contributors. The contract allows for token claims from the treasury, and includes admin functionality for managing roles and setting up airdrops. Auditing this smart contract is critical to validate the ERC20 functionalities the contract implement.

Auditing the key contracts of the AI Arena Protocol is essential to ensure the security of the protocol. These contracts form the backbone of the AI Arena protocol. Focusing on these contracts first will provide a solid foundation for understanding the protocol's operation and how it manages creation of different fighters, calculating battle results, increasing and decreasing a user's rewards.

In auditing these contracts, I examined for possible security risks like reentrancy, access control vulnerabilities, and logic inconsistencies. Moreover, I rigorously tested the functions and roles outlined in these contracts to ensure they operate as intended.

## System Overview

### Scope

- **FighterFarm.sol**: The FighterFarm contract manages the creation and ownership of AI Arena Fighter NFTs, allowing users to mint, claim, and reroll fighters with unique attributes. It includes mechanisms for staking fighters, transferring ownership, and interacting with the native ERC20 token (Neuron). The contract enforces a maximum number of fighters per address and rerolls per fighter, and integrates with external contracts for additional functionality. It adheres to the ERC721 standard for NFTs, with custom logic for fighter attributes and generation management.
-  **RankedBattle.sol**: The RankedBattle contract facilitates a staking-based reward system for the game where players can stake NRN tokens on fighters, participate in battles, and earn points based on battle outcomes. It tracks battle records, calculates staking factors, and distributes NRN rewards proportionally to the points earned. The contract includes mechanisms for staking, unstaking, claiming rewards, and managing rounds, with administrative functions to adjust game parameters and advance rounds. It interacts with other contracts like FighterFarm, VoltageManager, MergingPool, and StakeAtRisk to manage fighters, energy costs, point merging, and risked stakes, respectively.
-  **Neuron.sol**: The Neuron contract serves as an ERC20 token that embodies Neuron (NRN) tokens within the AI Arena's in-game ecosystem. It encompasses roles for minting, spending, and staking tokens, and initiates an initial minting process for the treasury and contributors. The contract enables token claims from the treasury and incorporates administrative features for role management and airdrop setup. Additionally, it facilitates mechanisms for minting new tokens up to a predefined maximum supply, burning tokens to decrease supply, and managing allowances for token transfers.
-  **AiArenaHelper.sol**: The AiArenaHelper contract is designed to manage the physical attributes of AI fighters in a the arena, with functionality to generate attributes based on fighter DNA and generation. It allows the owner to set and update probabilities and divisors for attribute generation, and includes ownership transfer capabilities. Special cases for certain icon types are handled, and dendroid fighters receive a fixed set of attributes. The contract also provides functions to add, delete, and retrieve attribute probabilities for different generations.
-  **FighterOps.sol**: The FighterOps library is designed for providing tools to manage Fighter NFTs with unique attributes. It includes functions to emit events upon fighter creation and to extract or view a fighter's detailed attributes. The library operates on Fighter structs, which encapsulate both physical and identifying characteristics of the NFTs.
-  **GameItems.sol**: The GameItems contract is an ERC1155-based smart contract for managing a collection of tradable and non-tradable game items with finite or infinite supply. It allows users to mint items by spending Neuron tokens (NRN), with daily minting allowances and the ability for admins to create new items and set their attributes. The contract includes functionality for transferring ownership, adjusting admin access, and burning items. It also enforces rules around item transferability and integrates with the Neuron contract for payment processing.
-  **MergingPool.sol**: The MergingPool contract allows users to earn new fighter NFTs by participating in a system where points are accumulated for fighters. Admins select winners each round based on external criteria, and winners can claim their NFT rewards. The contract tracks points, winners, and claims, and includes functions for ownership transfer and admin management. It integrates with the FighterFarm contract to mint NFTs for winners.
-  **StakeAtRisk.sol**: The StakeAtRisk contract manages the stakes (NRNs) that are at risk of being lost by fighters during ranked battles in a competition round. It allows the associated RankedBattle contract to update stakes at risk, reclaim stakes for winners, and set new rounds, during which it sweeps lost stakes to a treasury. Stake information is tracked per round and per fighter, with events emitted for key actions. Access to critical functions is restricted to the RankedBattle contract.
-  **VoltageManager.sol**: The VoltageManager contract manages a voltage system for a the game, allowing players to replenish their voltage using in-game items and spend voltage for certain actions. It includes ownership and admin controls to manage permissions. Voltage can be automatically replenished daily, and the contract interacts with the GameItems contract to burn items used for replenishment. Events are emitted for voltage changes to track player balances.

### Privileged Roles

There are several important roles in the protocol.

- **Owner**: This role is essential in every contract. It grants administrative permissions and control over critical functions. The holder of this role can manage other roles, configure contract parameters, and perform various administrative tasks.
- **Admin**: Admins have elevated privileges that allow them to adjust game settings. Admins are designated by the owner.
- **GameServer**: GameServer is an important role in the RankedBattle contract that updates the battle records for a player.
- **MINTER_ROLE - SPENDER_ROLE - STAKER_ROLE**: These are important roles that are set by the contract owner, that permits the minting,spending and staking of tokens. These roles are mainly present in the Neuron contract.

## Approach Taken-in Evaluating The AI Arena Protocol

Consequently, I conducted an analysis and audit of the subject protocol through the following steps:

1. **Core Protocol Contract Overview**: 

    My primary focus was to comprehensively comprehend the code base and offer recommendations for enhancing its functionality. The central objective was to examine the critical contracts and their interplay within the AI Arena Protocol.

    I started with the following contracts, which play crucial roles in the AI Arena protocol:

    **Main Contracts I Looked At this phase**

        FighterFarm.sol
        RankedBattle.sol
        Neuron.sol
        MergingPool.sol
        StakeAtRisk.sol

I started my analysis by examining the FighterFarm.sol contract which plays crucial role in the AI Arena protocol and manages the creation and ownership of the AI Arena fighter NFTs. After that I delved into the RankedBattle.sol contract which has the logic for the staking, unstaking and calculating the battle results. the Neuron.sol, MergingPool.sol and the stakeAtRisk.sol, were also thoroughly examined to ensure the secure flow of logic within the contracts.

2. **Documentation Review**: 
    Afterward, I proceeded to Review [these](https://docs.aiarena.io/gaming-competition) Docs for a more comprehensive and technical understanding of the AI Arena protocol.

3. **Compiling code and running provided tests**:
    I compiled the code and ran the provided tests.

4. **Manual Code Review**:
    In this phase, I conducted a line-by-line analysis.
    - **Line by Line Analysis**: Carefully observed the contract's intended functionality in contrast with its actual behavior, analyzing it line by line.

## Codebase Quality 

Overall, I consider the quality of the AI Arena protocol codebase to be Good. The code appears to be mature and well-developed. The implementation of the code adheres to various standards properly. 
Details are explained below:

| Codebase Quality Categories               | Comments
|------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability**  | The codebase demonstrates moderate maintainability with well-structured functions and comments, promoting readability. It exhibits reliability through defensive programming practices, parameter validation, and handling external calls safely. The use of internal functions for related operations enhances code modularity, reducing duplication. Libraries improve reliability by minimizing arithmetic errors. Adherence to standard conventions and practices contributes to overall code quality. However, full reliability depends on external contract implementations like openzeppelin.
| **Code Comments**                         | The contracts include comments that describes the purpose and operations of various sections, aiding developers and auditors in comprehending and managing the code. These comments offer insights into methods, variables, and the overarching architecture of the contracts. For instance, the "RankedBattle" contract's code comments furnish crucial details. They declare the title and the objective of the contract, explain all the variables and the functions within the contract.
| **Documentation**                         | The documentation of the AI Arena protocol is quite comprehensive and detailed, providing a solid overview of how AI Arena is structured and how its various aspects function. However, what the documentation does not provide is a sub part in the docs that explain each AI Arena smart contract in details, explaining the objectives of the contracts, explaining its methods, specific diagrams representing the internal interactions between the contracts, and how users interact with these contracts. Explaining the contracts with diagrams is a way to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. I am confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors.
| **Testing**                               | The overall line coverage percentage provided by tests is 90% which should be aimed at 100%.
| **Code Structure and Formatting**         | The codebase contracts are well-structured and formatted. It inherits from OpenZeppelin governance, and token standards in some contracts. The constructors initializes the contract with parameters and specified roles, contracts override functions, each component is provided with accompanying comments explaining their purpose. The code is organized, making it readable and maintainable.
| **Errors**                                | Custom errors should be used instead of require statements with string errors. Custom errors are easier to re-use and maintain.

## Systemic & Centralization Risks

### Systemic Risks:

1. In the FighterFarm contract, if rerollCost is too low, it may cause inflation by devaluing the Neuron token due to excessive rerolling. Conversely, if MAX_FIGHTERS_ALLOWED is too high, it could lead to player hoarding and market saturation, giving an unfair edge to certain players and potentially causing deflation by reducing token circulation. Both scenarios can disrupt the game's economy and player experience.
2. The FighterFarm contract allows for the incrementing of fighter generations and the number of rerolls allowed. If this functionality is mismanaged or exploited, it could lead to a loss of rarity and uniqueness of fighters, affecting their value and the overall balance of the game.
3. The FighterFarm contract relies on external contracts such as the AiArenaHelper, AAMintPass, and Neuron token. If any of these contracts have vulnerabilities, are manipulated, or fail, it could impact the FighterFarm contract's operations, potentially leading to loss of funds or NFTs.
4. In the RankedBattle contract the rankedNrnDistribution mapping determines the NRN token distribution for each round. If admins can arbitrarily change the distribution amount without checks or community consensus, it could lead to unfair reward allocation or inflationary pressure on the NRN token.
5. The contract RankedBattle relies on a designated _gameServerAddress to report battle outcomes. If this address is compromised or behaves maliciously, it could falsify battle records, leading to incorrect point and reward distribution.
6. The RankedBattle contract's economic parameters (e.g., bpsLostPerLoss, VOLTAGE_COST) need to be carefully balanced. If they are not set correctly, they could lead to an imbalanced game economy, where either too much risk or too little risk is associated with battles, affecting player engagement and the value of the NRN token.
7. In the Neuron contract while there is a MAX_SUPPLY limit, the contract allows for additional minting of tokens up to this cap. If the minter role is abused or if too many tokens are minted too quickly, it could lead to inflationary pressure on the NRN token.
8. In the MergingPool contract the pickWinner function allows admins to select winners for each round. If this power is abused or if the admin selection process lacks transparency, it could lead to unfairness and loss of trust among participants.

### Centralization Risks:

1. The contracts has an owner address (_ownerAddress) with exclusive rights to perform critical actions such as transferring ownership, incrementing fighter generations, adding staker addresses, setting contract instances for helper contracts, and updating the merging pool address, and many other actions in different contracts. If this address is an EOA it will be a centralized entity controlling the system.
2. In the RankedBattle contract admins can change critical game settings, such as the NRN distribution for each round and the basis points lost per loss. They can also advance the game to a new round. If admins act maliciously or carelessly, they could disrupt the game's economy and fairness, and is a centralization risk.
3. The contract RankedBattle relies on the _gameServerAddress which is a centralized entity to report battle outcomes. If compromised, it could lead to incorrect updates to battle records and points.
4. In the MergingPool contract there are no mechanisms within the contract for decentralized decision-making or community consensus. Changes to critical parameters and the selection of winners are at the discretion of the owner and admins.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the AI Arena protocol.**

## Conclusion

The AI Arena protocol showcases a robust and innovative gaming platform that effectively integrates AI with NFTs, offering users a unique experience in training and battling AI models. The key contracts, including FighterFarm.sol, RankedBattle.sol, and Neuron.sol, are well-structured and adhere to industry standards, ensuring a secure and functional ecosystem. However, the protocol faces systemic risks such as potential token inflation, alongside centralization risks stemming from the significant control wielded by owner and admin roles. To enhance the protocol's resilience and trustworthiness, it is recommended to implement decentralized governance mechanisms, introduce additional security practices, and consider further audits and bug bounty programs. These measures will help mitigate risks, optimize the gaming economy, and uphold the protocol's integrity.


### Time spent:
65 hours

### Time spent:
65 hours