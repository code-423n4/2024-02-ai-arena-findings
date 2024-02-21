### Comprehensive project overview
AI Arena is a blockchain-based PvP fighting game where players train AI fighters to compete in battles. 

These fighters are represented as NFTs, minted through the FighterFarm.sol smart contract, with unique physical attributes, generations, weight, elements, and types (regular Champion or Dendroid), alongside model data. 

The game's economy revolves around the native ERC20 token, $NRN, used for staking in ranked battles to earn rewards or face potential stakes loss based on performance. 

The RankedBattle.sol contract, having MINTER and STAKER roles, manages the reward system, updates battle results, and leverages a stakingFactor based on the square root of staked amounts for fair play. 

Additionally, players manage a "voltage" resource, necessary for engaging in battles, which replenishes daily or can be instantly refilled by purchasing batteries via the GameItems.sol contract. 

The core game mechanics, including battle outcomes and rewards distribution, operate off-chain, with the game server acting as the oracle. 

AI Arena blends NFT technology with traditional gaming to create a dynamic, strategy-based platform where players can earn cryptocurrency rewards through skilled play and strategic management of digital assets.


### Technical Documentation
Here is an analysis of all of the contracts and each of their functions.

1. **FighterFarm.sol:**

The **`FighterFarm`** contract for the AI Arena game manages the creation, ownership, and redemption of AI Arena Fighter NFTs. 

Below is a summary of each function within this contract:

- **`transferOwnership`**: Transfers the ownership of the contract to a new address. Only the current owner can call this function.
- **`incrementGeneration`**: Increases the generation count of a specified fighter type, which can only be called by the owner. It also increases the max rerolls allowed for that fighter type.
- **`addStaker`**: Adds a new address that is permitted to stake fighters, an action restricted to the owner.
- **`instantiateAIArenaHelperContract`**: Initializes the AI Arena Helper contract, a task that only the owner can perform.
- **`instantiateMintpassContract`**: Sets up the mint pass contract, again only callable by the owner.
- **`instantiateNeuronContract`**: Initializes the Neuron (ERC20 token) contract, a function reserved for the owner.
- **`setMergingPoolAddress`**: Assigns the address for the merging pool contract, exclusively callable by the owner.
- **`setTokenURI`**: Sets the token URI for a specific tokenId, a task that the delegated address can perform.
- **`claimFighters`**: Allows users to claim a predetermined number of fighters by verifying a signature from the delegated address.
- **`redeemMintPass`**: Burns mint passes to create fighter NFTs, ensuring all input arrays match in length and correspond to the same index across arrays.
- **`updateFighterStaking`**: Updates the staking status of a fighter, a function restricted to addresses with the staker role.
- **`updateModel`**: Updates the machine learning model for a fighter, callable only by the fighter's owner.
- **`doesTokenExist`**: Checks if a token ID exists within the contract.
- **`mintFromMergingPool`**: Mints a new fighter from the merging pool, restricted to being called by the merging pool address.
- **`transferFrom`** and **`safeTransferFrom`**: Transfer a fighter NFT from one address to another, with added checks for transfer eligibility.
- **`reRoll`**: Rerolls a fighter to randomly change its traits, restricted to the fighter's owner who must have sufficient $NRN and has not exceeded the reroll limit.
- **`contractURI`**: Provides the URI for the contract's metadata.
- **`tokenURI`**: Returns the metadata URI for a specific fighter token.
- **`supportsInterface`**: Indicates whether a given interface ID is supported by this contract, implementing ERC721 and ERC721Enumerable standards.
- **`getAllFighterInfo`**: Retrieves all relevant information about a specific fighter token.
- **`_beforeTokenTransfer`**: Hook that is called before any token transfer, implementing checks from ERC721 and ERC721Enumerable.
- **`_createFighterBase`**: Internally creates the base attributes for a new fighter based on its DNA and type.
- **`_createNewFighter`**: Internally creates a new fighter and mints the corresponding NFT to the designated address, utilizing custom or base attributes.
- **`_ableToTransfer`**: Checks whether a specific token is eligible for transfer, considering the max fighter limit per address and staking status.

2. **GameItems.sol**

This contract combines ERC1155 functionality for managing an array of game items with additional features like daily minting allowances, finite supplies, and integrated payment and pricing in $NRN tokens.

Below is a summary of each function in the **`GameItems`** contract:

- **`transferOwnership`**: Transfers the ownership of the contract to a new address, requiring the caller to be the current owner.
- **`adjustAdminAccess`**: Grants or revokes administrative privileges to a specific address, with the operation restricted to the contract owner.
- **`adjustTransferability`**: Modifies the transferability of a specific game item, enabling or disabling the ability to trade it. This is owner-restricted and emits corresponding **`Locked`** or **`Unlocked`** events.
- **`instantiateNeuronContract`**: Initializes the **`Neuron`** contract instance, setting its address for future interactions. Only the owner can call this.
- **`mint`**: Allows users to mint specified quantities of game items, charging them in $NRN tokens, with checks for item existence, sufficient balance, supply constraints, and daily minting allowances.
- **`setAllowedBurningAddresses`**: Authorizes specific addresses to burn game items, an action limited to administrators.
- **`setTokenURI`**: Assigns a unique URI to a game item token, enabling custom metadata to be set by administrators.
- **`createGameItem`**: Creates a new game item with specified attributes such as name, URI, supply information, and pricing. Only admins can call this function.
- **`burn`**: Enables authorized addresses to destroy specified amounts of game items, reducing their total supply.
- **`contractURI`**: Returns the URI for the contract metadata, a static function providing off-chain contract details.
- **`uri`**: Overrides the ERC1155 **`uri`** function to return custom URIs for each token, falling back to the default URI if a custom one isn't set.
- **`getAllowanceRemaining`**: Provides the remaining daily minting allowance for a specific game item for a user, reflecting how many more of the item the user can mint that day.
- **`remainingSupply`**: Reports the remaining supply of a specific game item, applicable to items with finite supplies.
- **`uniqueTokensOutstanding`**: Counts the total number of unique game item tokens managed by the contract.
- **`safeTransferFrom`**: Safely transfers game items from one address to another, with an added check to ensure the item is transferable.
- **`_replenishDailyAllowance`**: A private function that resets a user's daily minting allowance for a game item after a 24-hour interval.

3. **MergingPool.sol**

This contract interacts with **`FighterFarm`** for minting new fighters as rewards and uses internal mappings to manage points, claims, and administrative roles effectively.

- **`transferOwnership`**: Changes the owner of the contract to a new address, ensuring only the current owner can execute this action.
- **`adjustAdminAccess`**: Modifies administrative access for a given address, allowing only the contract owner to grant or revoke admin status.
- **`updateWinnersPerPeriod`**: Updates the number of winners allowed per competition period, restricted to administrators.
- **`pickWinner`**: Selects winners for the current round, ensuring the operation adheres to the expected winners per period and that selection has not been completed for the round. It resets winner points to zero and advances the round.
- **`claimRewards`**: Enables users to claim their rewards for multiple rounds, minting new fighters based on provided model URIs, types, and attributes. It updates the number of rounds claimed by the user.
- **`getUnclaimedRewards`**: Returns the number of unclaimed rewards for a specific user, calculating based on rounds not yet claimed by the user.
- **`addPoints`**: Adds points to a fighter's score in the merging pool, callable only by the ranked battle contract to ensure integrity in point allocation.
- **`getFighterPoints`**: Retrieves points for fighters up to a specified token ID, allowing for a bulk view of points allocated in the merging pool.

4. **Neuron.sol**

This contract implements an ERC20 token with extended functionalities for an in-game economy, such as roles for minting, spending, and staking, alongside mechanisms for token distribution and burning.

Here's a summary of each function in the **`Neuron`** contract:

- **`constructor`**: Initializes the contract, sets up initial roles, and mints the initial supply of NRN tokens to the treasury and contributors.
- **`transferOwnership`**: Transfers the contract's ownership to a new address, restricted to the current owner.
- **`addMinter`**: Grants the minter role to a new address, allowing it to mint new tokens, limited to the contract owner.
- **`addStaker`**: Assigns the staker role to a new address, enabling it to stake tokens, an action restricted to the owner.
- **`addSpender`**: Adds a new address to the spender role, permitting it to spend tokens on behalf of others, again limited to the owner's discretion.
- **`adjustAdminAccess`**: Modifies admin privileges for a specified address, controlled by the owner.
- **`setupAirdrop`**: Prepares an airdrop by setting allowances from the treasury to recipients, an action that can only be executed by admins.
- **`claim`**: Allows users to claim tokens from the treasury up to their approved allowance, emitting a **`TokensClaimed`** event upon success.
- **`mint`**: Mints new NRN tokens to a specified address, ensuring the action doesn't exceed the maximum supply and is performed by an address with the minter role.
- **`burn`**: Destroys a specified amount of tokens from the caller's balance, reducing the total supply.
- **`approveSpender`**: Approves a specified amount of the caller's tokens for spending by another account, requiring the caller to have the spender role.
- **`approveStaker`**: Similar to **`approveSpender`**, but for approving tokens for staking purposes, necessitating the caller to possess the staker role.
- **`burnFrom`**: Burns tokens from a specified account, dependent on having enough allowance from the token owner to the caller, effectively reducing the owner's balance and the caller's allowance.

5. **RankedBattle.sol**

The **`RankedBattle`** contract is designed for managing the staking, battling, and rewards distribution within the game ecosystem. Here's a summary of each function:

- **`transferOwnership`**: Changes the contract's ownership to a new address, enforceable by the current owner only.
- **`adjustAdminAccess`**: Grants or revokes admin privileges to an address, with this capability restricted to the contract's owner.
- **`setGameServerAddress`**: Assigns the game server's address for updating battle records, an action limited to the owner.
- **`setStakeAtRiskAddress`**: Sets the address of the **`StakeAtRisk`** contract and initializes its instance. This action is also restricted to the owner.
- **`instantiateNeuronContract`**: Initializes the **`Neuron`** (ERC20 token) contract instance, a function callable only by the owner.
- **`instantiateMergingPoolContract`**: Sets up the **`MergingPool`** contract instance, with the operation being restricted to the contract owner.
- **`setRankedNrnDistribution`**: Adjusts the NRN token distribution amount for the ranked battles of the current round, modifiable only by admins.
- **`setBpsLostPerLoss`**: Sets the basis points that players lose from their stake when they lose a battle, a parameter that only admins can adjust.
- **`setNewRound`**: Advances the game to a new round, enabling new claims and resetting battle and staking records, a function reserved for admin use.
- **`stakeNRN`**: Allows players to stake NRN tokens on their fighters, increasing their potential rewards from participating in battles.
- **`unstakeNRN`**: Permits players to withdraw their staked NRN tokens from their fighters, with certain restrictions to prevent abuse during ongoing rounds.
- **`claimNRN`**: Enables players to claim their earned NRN tokens based on their participation and success in ranked battles up to the current round.
- **`updateBattleRecord`**: Updates the battle record for a fighter based on the outcome of a battle, including win, tie, or loss. This function can only be called by the designated game server address.
- **`getBattleRecord`**: Retrieves the battle record (wins, ties, and losses) for a specific fighter token.
- **`getCurrentStakingData`**: Provides an overview of the current staking situation, including the round ID, the amount of NRN tokens to be distributed, and the total accumulated points for the round.
- **`getNrnDistribution`**: Returns the amount of NRN tokens distributed for winning in a particular round.
- **`getUnclaimedNRN`**: Calculates the amount of unclaimed NRN tokens for a specific player, based on their participation and success in previous rounds.
- **`_addResultPoints`** (Private): Internally calculates and assigns points to fighters based on battle outcomes, adjusting staking rewards and potentially transferring stakes to the risk pool based on the battle results.
- **`_updateRecord`** (Private): Internally updates a fighter's battle record after each match.
- **`_getStakingFactor`** (Private): Computes the staking factor for a fighter based on the square root of their staked amount, influencing the calculation of rewards.

6. **StakeAtRisk.sol**

The **`StakeAtRisk`** contract manages the NRNs (Neuron tokens) that players have at risk of losing during ranked battles in the AI Arena game. Here’s a breakdown of its functionality:

- **`Constructor`**: Initializes the contract by setting the treasury, Neuron (NRN), and RankedBattle contract addresses. It also instantiates the Neuron contract.
- **`setNewRound`**: Called by the RankedBattle contract to initiate a new round of competition. It sweeps all NRNs held in this contract to the treasury and updates the round ID. This function ensures that only the RankedBattle contract can call it.
- **`reclaimNRN`**: Allows the RankedBattle contract to reclaim NRN tokens from the stake at risk for a fighter who wins a match. It ensures the caller is the RankedBattle contract and that the fighter has sufficient stake at risk before transferring the reclaimed NRNs.
- **`updateAtRiskRecords`**: Called by the RankedBattle contract to place more NRN tokens at risk for a specific fighter, typically after a loss that results in a deficit of points. This function updates the stake at risk for the fighter and the total stake at risk for the round.
- **`getStakeAtRisk`**: Provides the amount of NRN tokens at risk for a specific fighter in the current round. This view function allows querying the stake at risk without altering the contract state.
- **`_sweepLostStake`** (Private): Transfers the lost stake (NRNs at risk that were not reclaimed by players) to the treasury at the end of a round. This internal function is part of the mechanism to reset the stakes at risk for a new round.

7. **VoltageManager.sol**

The **`VoltageManager`** contract oversees the voltage system in the AI Arena, regulating energy usage by players for initiating matches. Here's a summary of its functionality:

- **`Constructor`**: Initializes the contract, setting the owner and linking to the Game Items contract for managing in-game items like voltage batteries.
- **`transferOwnership`**: Allows the current owner to transfer ownership of the contract to a new address.
- **`adjustAdminAccess`**: Grants or revokes admin privileges to a specified address, ensuring only the owner can perform this action.
- **`adjustAllowedVoltageSpenders`**: Admins can authorize or deauthorize addresses to spend voltage on behalf of users, controlling who can initiate actions that consume voltage.
- **`useVoltageBattery`**: Players can replenish their voltage to the maximum limit by using a voltage battery, an in-game item. This action burns a battery from the player's inventory.
- **`spendVoltage`**: Deducts a specified amount of voltage from a player's total. This function can be called directly by the player or by an approved spender, facilitating various gameplay mechanics that require energy expenditure.
- **`_replenishVoltage`** (Private): Automatically replenishes a player's voltage to full capacity every 24 hours, ensuring players have the energy needed to participate in daily activities.

8. **AiArenaHelper.sol**

The **`AiArenaHelper`** contract assists in generating and managing physical attributes for AI Arena fighters. Here’s a breakdown of its functionality:

- **`Constructor`**: Initializes the contract with predefined attribute probabilities for the initial generation of fighters. It sets the owner and maps each attribute to a divisor for determining their rarity.
- **`transferOwnership`**: Allows the current owner to transfer ownership of the contract to a new address.
- **`addAttributeDivisor`**: Adds divisors for attributes, which are used to calculate the rarity of each attribute of a fighter. This function is restricted to the owner.
- **`createPhysicalAttributes`**: Generates physical attributes for a fighter based on its DNA, generation, and special icons type. This function considers whether the fighter is a dendroid and uses custom logic for determining attributes based on the fighter's DNA and predefined attribute probabilities.
- **`addAttributeProbabilities`**: Allows the owner to add attribute probabilities for a new generation of fighters, enabling customization of fighter attributes as the game evolves.
- **`deleteAttributeProbabilities`**: Permits the owner to delete attribute probabilities for a given generation, providing flexibility in managing the game's content and fighter generation logic.
- **`getAttributeProbabilities`**: Publicly accessible function that returns the attribute probabilities for a specified generation and attribute, facilitating transparency and enabling external calculations related to fighter attributes.
- **`dnaToIndex`**: Converts a fighter’s DNA and rarity rank into an attribute probability index, determining the rarity and type of attributes a fighter possesses based on its genetic code.

9. **FighterOps.sol**

**`FighterOps`**, is a Solidity library designed for managing Fighter NFTs within the AI Arena game. It provides a structured way to define, emit events for, and retrieve details about fighters.

Here’s a breakdown of its functionality:

- **`fighterCreatedEmitter`**: Emits the **`FighterCreated`** event, signaling the creation of a new fighter with its core attributes.
- **`getFighterAttributes`**: Extracts and returns the physical attributes of a fighter as an array, making it easier to access these details for game mechanics or display purposes.
- **`viewFighterInfo`**: Compiles and returns comprehensive information about a fighter, including the **`owner`** address and a variety of attributes and identifiers. This function is essential for external calls to fetch detailed data about a fighter for gameplay or display.

### Architecture Overview

#### Key Components of the AI Arena Protocol

**Fighter NFTs**: Managed by the FighterFarm.sol contract, each fighter NFT encapsulates the fighter's physical attributes, generation, weight, element, fighter type (regular Champion or Dendroid), and model data (model type and model hash).

**Ranked Battles**: The RankedBattle.sol contract allows players to stake $NRN tokens on their fighters and participate in ranked battles. Winners earn rewards based on battle outcomes and the amount staked. The contract uses a staking factor calculated from the square root of the staked amount to determine points after each match.

**Voltage System**: Managed by the VoltageManager.sol contract, voltage acts as an energy system for fighters. Players must manage their fighters' voltage to participate in battles, with options to wait for natural replenishment or purchase batteries for instant recharge.

**Reward Mechanism**: Rewards are distributed in $NRN tokens, the platform's native ERC20 token. The Neuron.sol contract, along with StakeAtRisk.sol for managing stakes at risk, and MergingPool.sol for potential earnings of new fighter NFTs, supports the reward system.

**Game Items**: The GameItems.sol contract will represent various in-game items, for now, only batteries for voltage replenishment are available.

#### User Flows

**Fighter NFT Acquisition**: Players can acquire fighter NFTs through minting in the FighterFarm.sol contract or by trading on the marketplace.

**Preparation for Battle**: Players stake NRN tokens on their fighters, manage their voltage, and may equip them with in-game items for enhanced performance.

**Participation in Ranked Battles**: Players enter their staked fighters into battles, with outcomes determined off-chain by the game server, acting as an oracle.

**Reward Distribution**: Post-match, rewards in NRN tokens are calculated and distributed based on the battle outcomes, staked amounts, and points system. Players can also claim their stakes or any earnings from the MergingPool.sol.

Here is a non-exhaustive diagram of the interactions between the contracts that I created and used for reference throughout the contest:
![AIArenaArchitecture](https://img001.prntscr.com/file/img001/Uv6yAwKjTrOL5WLQPud_yQ.png)
### Centralization/Systemic risks
The reliance on off-chain servers to determine the outcomes of matches and interactions within the game introduces a level of trust and centralization. 

The server acts as an oracle, and if it is compromised or malfunctions, it could affect the integrity of the game's outcomes and rewards.

### Approach taken

Day 1-3: Initial Familiarization and High-Level Overview
- Gain a comprehensive understanding of the AI Arena protocol's purpose and structure.
- Review all smart contracts briefly to develop a high-level mental model of their interactions and the overall architecture.

Day 4-7: In-Depth Contract Review
- Start a detailed review of each contract, focusing on core components like FighterFarm, RankedBattle, and MergingPool.
- Examine user roles, permissions, and typical interactions within the system.
- Follow user flows through the system, including minting and claiming fighters, staking and unstaking NRN, buying batteries, and claiming rewards
- Verify the flows against the intended functionality outlined in the documentation.

Day 8-10: Test Case Development and Execution
- Develop and run custom test cases, particularly for edge cases and potential security vulnerabilities.
- Test the integration and interaction of different contracts within the ecosystem.


Day 11-12: Final Review and Issue Compilation
- Conduct a final comprehensive review of all contracts and user flows.
- Compile and categorize identified issues based on severity and impact.
- Prepare a detailed analysis report, including findings, recommendations, and suggestions for improvements.

### Conclusion
The AI Arena protocol introduces a unique blend of AI-driven gameplay and blockchain technology, offering an engaging PvP experience through NFTs and token staking. 

It capitalizes on the growing interest in decentralized gaming, allowing players to train, battle, and earn within a secure ecosystem. 

By tokenizing fighters and implementing a reward system based on competitive outcomes, it creates a dynamic economy that rewards skill, strategy, and participation.

### Time spent:
50 hours