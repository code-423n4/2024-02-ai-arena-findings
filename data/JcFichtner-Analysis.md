## 1 - AI Arena: Project Overview

AI Arena is a blockchain-based player versus player (PvP) fighting game platform that integrates various elements of web3 technology, NFTs, and cryptocurrency into its gameplay and reward systems. 

## 2 - key components and concepts 

1 - ***AI Fighters and Tokenization:*** 

Players own AI fighters that are represented as NFTs. These fighters are trained by humans and have specific attributes (physical appearance, generation, weight, element, and fighter type) and model data (model type and model hash) embedded within their NFT data. The smart contract responsible for this is called FighterFarm.sol.

2 - ***Fighter Attributes:***

. Physical Attributes & Generation: Affect the visual appearance of the fighters.

. Weight: Influences the fighter's battle capabilities.

. Element: Determines the special abilities the fighter possesses.

. Fighter Type: Distinguishes between regular Champions and Dendroids, likely 
  affecting    gameplay strategies and abilities.
                    
. Model Data: Includes information necessary for running the AI model, such as 
  its type and a unique identifier (hash).  

3 - ***Gameplay and Rewards:*** 

Players can enter their NFT fighters into ranked battles to earn rewards in the game's native ERC20 cryptocurrency, denoted as NRN. The Neuron.sol smart contract defines this token. Participation and performance in these battles can lead to earning or losing NRN, with rewards facilitated by the RankedBattle.sol smart contract, which has special roles to mint (create new tokens) and stake (lock up tokens to participate).

4 - ***In-game Economics:***

. Staking and Rewards: Players earn rewards by staking $NRN and winning matches. 
  The stake's size influences the rewards, adjusted by a staking factor 
  calculated from the square root of the staked amount.

. SPENDER Role: The FighterFarm.sol and GameItems.sol smart contracts can spend 
  $NRN on behalf of players for in-game purchases, utilizing a designated SPENDER 
  role.

5 - ***Voltage System:*** 

This is a unique energy system where each player's wallet has a "voltage" level that depletes with each ranked battle. It replenishes over time or can be instantly refilled by purchasing a battery through the GameItems.sol smart contract.

6 - ***Off-Chain Game Logic:*** 

While the game utilizes blockchain technology for NFTs, cryptocurrency, and certain game mechanics, the core game logic, including the outcome of ranked matches, is executed off-chain on the game's servers. These servers act as oracles, providing authoritative results for the matches that affect the blockchain-recorded elements like rewards distribution.

In summary, this game combines traditional gaming mechanics with blockchain technology to create a decentralized, competitive platform where players can train, battle, and earn rewards with AI-controlled NFT fighters. The game leverages smart contracts for fighter attributes, in-game transactions, and reward systems, while relying on off-chain servers for core gameplay decisions.

## 3 - Overview of the Contracts (SCOPE)

###### Explanation for AiArenaHelper.sol

AiArenaHelper is intended to generate and manage the physical attributes of fighters in an AI Arena game. The contract includes functionality for ownership management, attribute probability configuration, and the creation of fighter attributes based on DNA.

***Here's a breakdown of the code with comments on functionality:***

1. **Version Pragma**: The contract specifies that it is compatible with Solidity 
     versions from 0.8.0 to less than 0.9.0. This is a good practice as it 
     prevents 
     the contract from being compiled with a potentially incompatible compiler 
     version.

2. **Import Statement**: The contract imports `FighterOps` library, which contain 
     the `FighterPhysicalAttributes` struct used later in the code.

3. **State Variables**:

   - `attributes`: A fixed array of strings representing different physical 
      attributes of a fighter.

   - `defaultAttributeDivisor`: An array of divisors used to calculate attribute 
      rarity from a fighter's DNA.

   - `_ownerAddress`: A private variable that stores the address of the contract 
      owner.

4. **Mappings**:

   - `attributeProbabilities`: A nested mapping that stores arrays of 
      probabilities for each attribute based on the fighter generation.

   - `attributeToDnaDivisor`: A mapping that associates each attribute with a 
      divisor for DNA calculations.

5. **Constructor**:

   - Initializes the owner address to the deployer of the contract.

   - Calls `addAttributeProbabilities` to set up the initial probabilities for 
     generation 0.

   - Sets the DNA divisors for each attribute.

6. **Ownership Transfer**:

   - `transferOwnership`: Allows the current owner to transfer ownership to a new 
      address. It has a security check to ensure only the current owner can call it.

7. **Attribute Divisor Management**:

   - `addAttributeDivisor`: Allows the owner to add new attribute divisors. It 
      checks that the input array length matches the number of attributes.

8. **Attribute Generation**:

   - `createPhysicalAttributes`: Generates physical attributes for a fighter 
      based on DNA, generation, icons type, and dendroid status. It has special 
      cases for certain icon types and returns fixed attributes for dendroids.

9. **Attribute Probability Management**:

   - `addAttributeProbabilities`: Allows the owner to add attribute probabilities 
      for a given generation.

   - `deleteAttributeProbabilities`: Allows the owner to delete attribute 
      probabilities for a given generation.

   - `getAttributeProbabilities`: Returns the attribute probabilities for a given 
      generation and attribute.

   - `dnaToIndex`: Converts DNA and rarity rank into an attribute probability 
      index.

###### Explanation for FighterFarm.sol

FighterFarm is an ERC721 token representing AI Arena Fighter NFTs. The contract includes functionality for minting, transferring, and managing these NFTs, including staking and rerolling fighters with different attributes. It inherits from OpenZeppelin's `ERC721` and `ERC721Enumerable` contracts.

***Here's a breakdown of the code with comments on functionality and potential security concerns:***

1. **Imports and Contract Declaration**: The contract imports several other 
     contracts and libraries, including OpenZeppelin's ERC721 contracts. It 
     defines the `FighterFarm` contract, which inherits from `ERC721` and 
    `ERC721Enumerable`.

2. **Events**: The contract defines events for locking and unlocking fighters, 
     which are used to indicate whether a fighter can be traded.

3. **State Variables**: The contract declares several state variables, including 
     constants, arrays, and mappings to track various aspects of fighters and 
     their owners.

4. **Constructor**: The constructor sets the initial owner, delegated address, 
     and treasury address. It also initializes the `numElements` mapping for 
     generation 0.

5. **Ownership and Role Management**: Functions like `transferOwnership`, 
    `addStaker`, and others are used to manage ownership and roles. These 
     functions are restricted to the owner of the contract.

6. **Contract Instantiation**: Functions to instantiate related contracts like 
    `AiArenaHelper`, `AAMintPass`, and `Neuron` are provided. These are also 
     restricted to the owner.

7. **Minting and Claiming**: The contract includes functions to claim fighters 
     (`claimFighters`) and redeem mint passes (`redeemMintPass`). The 
     `claimFighters` function requires a signature verification to ensure claims 
     are authorized.

8. **Staking**: The `updateFighterStaking` function allows stakers to lock or 
     unlock fighters for trading.

9. **Model Updates**: Fighters' machine learning models can be updated using the 
    `updateModel` function, which is restricted to the fighter's owner.

10. **Token Existence**: The `doesTokenExist` function checks if a token exists.

11. **Minting from Merging Pool**: The `mintFromMergingPool` function allows 
      minting new fighters from a merging pool, restricted to the merging pool 
      address.

12. **Transfer Overrides**: The `transferFrom` and `safeTransferFrom` functions 
      are overridden to include a custom check for the ability to transfer 
      fighters.

13. **Rerolling Fighters**: The `reRoll` function allows owners to reroll 
      fighters' attributes at a cost, with restrictions on the number of rerolls 
      and the owner's balance of the `Neuron` token.

14. **Metadata and Interface Support**: Functions like `contractURI`, `tokenURI`, 
      and `supportsInterface` provide metadata and interface support.

15. **Internal and Private Functions**: The contract includes internal and 
      private functions for token transfer hooks, creating fighter bases, minting 
      new fighters, and checking transfer eligibility.

###### Explanation for FighterOps.sol

FighterOps is a library intended to be used for managing Fighter NFTs. 

***Here's a breakdown of the code and its functionality:***

1. **Version Pragma**: The code specifies that it is compatible with Solidity 
     versions from 0.8.0 to less than 0.9.0. This is a good practice as it 
     prevents the code from being compiled with a compiler version that could 
     introduce breaking changes or unknown bugs.

2. **Library Definition**: `FighterOps` is a library, which means its functions 
     can be called by other contracts. Libraries are typically used for reusable 
     code and can help save gas when deployed correctly.

3. **Events**: The `FighterCreated` event is declared, which logs the creation of 
     a new Fighter NFT with its `id`, `weight`, `element`, and `generation`.

4. **Structs**: Two structs are defined, `FighterPhysicalAttributes` and 
    `Fighter`. These structs are used to represent the physical attributes of a 
     fighter and the fighter itself, including metadata like `modelHash` and 
    `modelType`.

5. **Public Functions**:

   - `fighterCreatedEmitter`: This function emits the `FighterCreated` event. It 
      is marked as `public`, which means it can be called by anyone. However, 
      this function should likely be `internal` or have access controls to 
      prevent unauthorized users from emitting events arbitrarily, which could 
      lead to event spamming or mislead off-chain services that listen to these 
      events.

   - `getFighterAttributes`: This function returns an array of the fighter's 
      physical attributes. It is marked as `public view`, which is appropriate 
      for a read-only function that does not modify state.

   - `viewFighterInfo`: This function returns comprehensive information about a 
      fighter, including the owner's address and various attributes of the 
      fighter. It is also marked as `public view`.

###### Explanation for GameItems.sol

The Solidity code provided defines a smart contract named `GameItems` that inherits from OpenZeppelin's `ERC1155` standard, which is a multi-token standard allowing for the creation of fungible and non-fungible tokens within a single contract. The contract is designed to manage game items for a platform called AI Arena.

***Here's a breakdown of the contract's functionality:***

1. **Events**: The contract defines events for buying items, locking, and 
     unlocking them, which are emitted when these actions occur.

2. **Structs**: A `GameItemAttributes` struct is defined to hold attributes for 
     each game item, such as name, supply type, transferability, remaining items, 
     price, and daily allowance.

3. **State Variables**: The contract has several state variables, including the 
     contract's name and symbol, an array of all game items, treasury and owner 
     addresses, a count of items, and an instance of another contract called ` 
     ´Neuron`.

4. **Mappings**: There are mappings to track allowances, replenish times, allowed 
     burning addresses, admin status, and token URIs.

5. **Constructor**: The constructor sets the initial owner and treasury addresses 
     and marks the owner as an admin.

6. **Ownership Transfer**: The contract allows the owner to transfer ownership, 
     adjust admin access, and set the transferability of game items.

7. **Minting**: The `mint` function allows users to mint game items by paying 
     with the `Neuron` token. It checks for sufficient balance, supply 
     constraints, and daily allowances.

8. **Admin Functions**: Admins can set burning addresses, token URIs, and create 
     new game items with specific attributes.

9. **Burning**: Certain addresses are allowed to burn game items.

10. **URI Management**: The contract overrides the `uri` function to return 
      custom URIs for tokens and provides a `contractURI` for the contract 
      metadata.

11. **Allowance and Supply Queries**: Functions are provided to check the 
      remaining daily allowance and supply of game items.

12. **Transfer Override**: The `safeTransferFrom` function is overridden to check 
      if an item is transferable before allowing the transfer.

###### Explanation for MergingPool.sol

MergingPool is part of a system that allows users to earn new fighter NFTs. The contract interacts with another contract called `FighterFarm` and is designed to work with a ranked battle system. 

***Here's a breakdown of the contract's functionality:***

1. **Ownership and Admin Management:**

   - The contract has an owner (`_ownerAddress`) who can transfer ownership and 
     adjust admin access (`isAdmin` mapping).

   - Admins can change the number of winners per period (`winnersPerPeriod`).

2. **Round Management:**

   - The contract keeps track of rounds (`roundId`) and whether the selection of 
     winners for a round is complete (`isSelectionComplete` mapping).

3. **Points System:**

   - Points can be added to fighters (`fighterPoints` mapping) by the ranked 
     battle contract (`_rankedBattleAddress`).

   - The total points across all fighters are tracked (`totalPoints`).

4. **Winner Selection:**

   - Admins can pick winners for the current round (`pickWinner` function), 
     ensuring the number of winners matches `winnersPerPeriod` and that winners 
     haven't been selected for the round yet.

   - Winners' points are reset to zero, and their addresses are recorded 
     (`winnerAddresses` mapping).

5. **Claiming Rewards:**
   - Users can claim rewards for multiple rounds (`claimRewards` function), which 
     calls the `mintFromMergingPool` function on the `FighterFarm` contract to 
     mint new NFTs.

   - The contract keeps track of the number of rounds a user has claimed 
     (`numRoundsClaimed` mapping).

6. **Querying:**

   - Users can check the number of unclaimed rewards (`getUnclaimedRewards` 
     function).

   - The contract provides a way to retrieve fighter points up to a specified 
     token ID (`getFighterPoints` function).

###### Explanation for Neuron.sol

Neuron is an ERC20 token with additional features for role-based access control. The contract uses OpenZeppelin's `ERC20` and `AccessControl` contracts as base contracts. 

***Here's a breakdown of the code with comments:***

1. **Version Pragma and Imports**: The contract specifies that it is compatible with Solidity versions from 0.8.0 to below 0.9.0 and imports ERC20 and AccessControl functionality from OpenZeppelin, which are well-audited and secure libraries.

2. **Contract Definition**: The `Neuron` contract inherits from `ERC20` and `AccessControl`. It is intended to represent Neuron (NRN) tokens used in AI Arena's in-game economy.

3. **Events**: Two custom events, `TokensClaimed` and `TokensMinted`, are defined for logging token claims and mints.

4. **State Variables**: The contract defines roles for minting, spending, and staking tokens using keccak256 hashes. It also sets initial mint amounts for the treasury and contributors, as well as a maximum supply cap.

5. **Constructor**: The constructor sets the initial owner, mints the initial supply to the treasury and contributor addresses, and sets the owner as an admin.

6. **Ownership and Role Management**: Functions are provided to transfer ownership and add addresses to various roles (minter, spender, staker). These functions can only be called by the owner, which is a single point of failure and centralization risk.

7. **Admin Access**: An `isAdmin` mapping is used to grant admin privileges, which can be adjusted by the owner. This is another centralization risk, as the owner has significant control over the contract.

8. **Airdrop Setup**: An `isAdmin`-protected function allows setting up airdrops by approving token allowances from the treasury to recipients. This function could be abused if an admin's account is compromised.

9. **Token Claiming**: Users can claim tokens from the treasury if they have an allowance, which emits a `TokensClaimed` event. This function relies on the correct setup of allowances and could be a vector for abuse if not managed properly.

10. **Minting and Burning**: The contract allows minting and burning of tokens, with checks for the maximum supply and appropriate roles. The mint function should ensure that the total supply plus the minted amount does not exceed the maximum supply, but the comparison should use `<=` instead of `<` to allow minting up to the max supply.

11. **Role-Based Approvals**: Functions are provided for spenders and stakers to approve allowances, which are protected by role checks.
12. **Burning from Allowance**: The `burnFrom` function allows burning tokens from an account with sufficient allowance. It correctly updates the allowance after burning.

###### Explanation for RankedBattle.sol

RankedBattle is intended to manage the staking system for the game. Players can stake NRN tokens on fighters, participate in battles, and earn rewards based on the outcomes of these battles. The contract includes functionality for staking and unstaking tokens, claiming rewards, and updating battle records.

***Here's a summary of the contract's functionality, along with comments:***

1. **Contract Initialization**: The constructor sets up the contract with initial values and addresses for the owner, game server, and other related contracts like `FighterFarm`, `VoltageManager`, etc.

2. **Ownership and Admin Management**: Functions are provided to transfer ownership, adjust admin access, and set various addresses. These functions are protected by `require` statements that check if the caller is the owner or an admin, which is a standard security practice.

3. **Staking and Unstaking**: The contract allows users to stake and unstake NRN tokens on fighters. It checks for ownership of the fighter and whether the user has enough tokens to stake. It also prevents staking after unstaking in the same round, which is a good practice to prevent certain types of gaming the system.

4. **Claiming Rewards**: Users can claim their accumulated rewards based on the points they've earned in battles. The contract ensures that users can only claim once per round.

5. **Battle Record Management**: The contract allows updating battle records, which includes spending voltage for initiating battles and calculating points based on the battle outcome and ELO factor. Only the game server address can update battle records, which is a centralized point of control and could be a potential security risk if the game server is compromised.

6. **Round Management**: Admins can advance the round, which affects the claiming process and the distribution of rewards.

7. **Events**: The contract emits events for staking, unstaking, claiming, and points changes, which is good for transparency and off-chain tracking.

###### Explanation for StakeAtRisk.sol

StakeAtRisk interacts with the token contract (`Neuron`) and the game contract (`RankedBattle`). This contract manages tokens (referred to as NRNs) that are at risk of being lost during a round of competition in a game.

***Here's a breakdown of the contract's functionality:***

1. **State Variables and Constructor:**

   - `roundId`: Tracks the current round of the game.

   - `treasuryAddress`: The address where lost stakes are sent.

   - `totalStakeAtRisk`: A mapping from `roundId` to the total amount of NRNs at risk for that round.

   - `stakeAtRisk`: A nested mapping from `roundId` to `fighterId` to the amount of NRNs at risk for that fighter.

   - `amountLost`: Tracks the amount of NRNs swept from each address.

   - The constructor sets the treasury address, the Neuron contract instance, and the RankedBattle contract address.

2. **Events:**

   - `ReclaimedStake`: Emitted when NRNs are reclaimed from the contract.

   - `IncreasedStakeAtRisk`: Emitted when more NRNs are placed at risk.

3. **External Functions:**

   - `setNewRound`: Called by the RankedBattle contract to start a new round and sweep lost stakes to the treasury.

   - `reclaimNRN`: Allows the RankedBattle contract to reclaim NRNs for a winning fighter.

   - `updateAtRiskRecords`: Called by the RankedBattle contract to increase the stake at risk for a losing fighter.

   - `getStakeAtRisk`: Returns the amount of NRNs at risk for a specific fighter in the current round.

4. **Private Functions:**

   - `_sweepLostStake`: Transfers the lost stake to the treasury contract.

###### Explanation for VoltageManager.sol

VoltageManager is part of the game ecosystem for managing a resource called "voltage" within the game. 

***Here's a breakdown of what the code does:***

1. **Contract Initialization**: The contract is initialized with an owner address and a reference to a `GameItems` contract. The owner is also set as an admin.

2. **Ownership Transfer**: The contract allows the current owner to transfer ownership to a new address.

3. **Admin Management**: The owner can grant or revoke admin privileges to other addresses.

4. **Voltage Spender Management**: Admins can authorize or deauthorize other addresses to spend voltage on behalf of the owner.

5. **Voltage Battery Usage**: Any user can use a voltage battery to replenish their voltage to 100, provided they have at least one of the required game item (with ID 0) and their current voltage is less than 100. The game item is then burned (removed from the user's balance).

6. **Voltage Spending**: Users can spend voltage, which is deducted from their balance. If the user's voltage replenish time has passed, their voltage is first replenished to 100, and the replenish time is reset to one day in the future.

7. **Voltage Replenishment**: A private function `_replenishVoltage` is used to set a user's voltage to 100 and their voltage replenish time to one day from the current time.

## 4 - Security Considerations


###### 1 - AiArenaHelper.sol

**Potential Security Concerns:**

- **Centralization of Control**: The contract is highly centralized, with the owner having exclusive control over critical functions like `transferOwnership`, `addAttributeDivisor`, `addAttributeProbabilities`, and `deleteAttributeProbabilities`. This centralization could be a point of failure if the owner's address is compromised.

- **Lack of Events**: The contract does not emit events for ownership transfer or changes in attribute probabilities/divisors. Events are crucial for transparency and off-chain tracking of changes.

- **Magic Numbers**: The contract uses magic numbers like `50` in `createPhysicalAttributes`, which can be confusing and error-prone. It's better to define these as named constants.

- **Input Validation**: The contract assumes that the input to the constructor and `addAttributeDivisor` is correct in length and content. Malformed input could lead to unexpected behavior.

- **Reentrancy**: There are no external calls to untrusted contracts, so reentrancy is not a concern here.

- **Integer Overflow/Underflow**: Since Solidity 0.8.0 and above have built-in overflow/underflow checks, this is not a concern unless explicitly using unchecked blocks.

###### 2 - FighterFarm.sol


**Potential Security Concerns:**

- **Centralization Risks**: The contract has a high degree of centralization, with many functions restricted to the owner. This could be a single point of failure or abuse.

- **Signature Verification**: The `claimFighters` function relies on signature verification, which is secure as long as the private key for signing is kept safe. If the key is compromised, unauthorized claims could be made.

- **Reentrancy**: The `reRoll` function calls an external contract (`_neuronInstance.transferFrom`). If the `Neuron` contract is not reentrancy-safe, this could be a vulnerability. However, the code does not show any immediate reentrancy concerns.

- **ERC721 Compliance**: The contract should ensure that it remains compliant with the ERC721 standard, especially when overriding functions.

- **Access Control**: The contract uses a simple access control mechanism based on address equality. More sophisticated access control (e.g., using OpenZeppelin's `AccessControl` contract) could provide more flexibility and security.

- **Input Validation**: The contract should validate inputs, especially array lengths and indices, to prevent out-of-bounds access or other issues.

###### 3 -  FighterOps.sol


**Potential Security Flaws**:

- **Access Control**: The `fighterCreatedEmitter` function lacks access control, which means that any user can call it and emit the `FighterCreated` event. This could be exploited to emit false events.

- **Storage Pointer**: The `getFighterAttributes` and `viewFighterInfo` functions take a `Fighter storage self` parameter. This means they are intended to be called with a reference to a `Fighter` struct stored in a calling contract's state. 

- **Gas Optimization**: While not a security flaw, using a library for emitting events or returning static information from storage might not be the most gas-efficient approach. Inlining these functionalities in the main contract could save gas, as external library calls have additional gas overhead.

**Additional Notes**:

- The code does not implement any actual logic for creating or managing fighters; it only provides utility functions and an event emission function.

- The `viewFighterInfo` function includes an `owner` parameter, but it does not check if the `owner` is related to the `Fighter` struct. This could be misleading if the `owner` parameter does not actually own the `Fighter` in question.

###### 4 - GameItems.sol


**Potential Security Flaws and Considerations:**

- **Centralization Risks**: The contract has a single owner who has significant control over the contract, including the ability to transfer ownership, adjust admin access, and change transferability of items. This centralization could be a risk if the owner's address is compromised.

- **Admin Powers**: Admins have the power to create items, set URIs, and allow burning addresses. If an admin account is compromised, it could lead to unauthorized creation or modification of game items.

- **Reentrancy**: The `mint` function calls an external contract (`_neuronInstance.transferFrom`) before updating the state. This could potentially be a reentrancy vulnerability if the `Neuron` contract is not reentrancy-safe.

- **Daily Allowance Replenishment**: The daily allowance replenishment logic could be exploited if the `dailyAllowanceReplenishTime` is not properly managed or if there's a way to manipulate the block timestamp.

- **Finite Supply Checks**: The contract checks for finite supply during minting, but there's no explicit cap on the `_itemCount`, which could potentially overflow if not managed correctly.

- **Access Control**: The contract uses a simple boolean mapping for admin access control, which is less flexible and secure compared to using a role-based access control system like OpenZeppelin's `AccessControl`.

- **Token URI Privacy**: The `_tokenURIs` mapping is private, which means that there's no direct way for other contracts to access the token URIs if needed.

- **Approval Pattern**: The `mint` function uses an approval pattern where the user must approve the contract to spend their `Neuron` tokens. This pattern is generally safe but requires users to make two transactions and understand the approval mechanism.

- **Error Handling**: The contract relies on `require` statements for error handling, which is standard, but it could include more descriptive error messages for better user experience.

###### 5 - MergingPool.sol


 **Security Considerations:**

  **Access Control:** 

   - The contract uses `require` statements to restrict certain functions to the owner or admins, which is a standard practice for access control.

   - However, there is no use of OpenZeppelin's `Ownable` or `AccessControl` contracts, which are well-tested and commonly used for managing ownership and permissions.

 - **Potential Centralization:** The contract relies heavily on centralized control by the owner and admins, which could be a point of failure or abuse.

 - **Input Validation:** The `pickWinner` function checks for the correct number of winners and whether the selection is complete for the round, which is good practice to prevent incorrect state changes.

 - **Reentrancy:** There are no external calls to untrusted contracts within functions that modify state, reducing the risk of reentrancy attacks.

- **Integer Overflow/Underflow:** Since the contract is using Solidity ^0.8.0, it is protected from overflow and underflow by default due to built-in checks.

- **DoS by Block Gas Limit:** The `claimRewards` function iterates over rounds and winners, which could potentially run into block gas limit issues if there are many rounds or winners to process.

- **Visibility of Functions:** The `addPoints` function is marked as `public`, which is appropriate since it is called by another contract. However, it could be marked as `external`.

- **Array Initialization:** In the `getFighterPoints` function, the array `points` is initialized with a length of 1, but it should be initialized with the length of `maxId` to avoid out-of-bounds errors.

- **Event Emission:** The contract emits events for significant state changes (`PointsAdded`, `Claimed`), which is good practice for transparency and off-chain tracking.

- **No Selfdestruct or Unsafe Operations:** The contract does not contain a `selfdestruct` function or other unsafe operations, which is a positive sign for contract stability.

- **Lack of Time Constraints:** There are no time-based conditions for rounds or claiming, which could be a design choice but might also lead to unexpected user experiences if not handled in the frontend application.

###### 6 - Neuron.sol


**Security Concerns and Recommendations**:

- **Centralization Risk**: The contract is highly centralized, with the owner having the power to manage roles and admin access. This could be a single point of failure if the owner's account is compromised.

- **Role Management**: The contract should ideally use OpenZeppelin's role management functions for consistency and security.

- **Minting Logic**: The mint function should allow minting up to the maximum supply, not strictly less than it.

- **Airdrop Function**: The `setupAirdrop` function could be a risk if an admin is compromised. Consider implementing a more decentralized approach or additional checks.

- **Upgradeability**: The contract does not appear to be upgradeable. If any issues are found or improvements are needed, the contract cannot be easily updated.

###### 7 - RankedBattle.sol


 **Security Considerations**:

   - The contract relies on external contracts like `Neuron`, `FighterFarm`, `VoltageManager`, `StakeAtRisk`, and `MergingPool`. It's crucial that these contracts are also secure, as vulnerabilities in any of them could affect the `RankedBattle` contract.

   - The use of `require` statements for access control is standard, but the contract could benefit from using a modifier for repeated owner-only checks.

   - The contract uses a library `FixedPointMathLib` for calculating the square root, which is a good practice to ensure mathematical operations are handled correctly.

   - The contract does not appear to have a withdrawal function for the contract owner to extract tokens, which is a positive sign for users' security, as it reduces the risk of rug pull.

   - The contract does not implement any emergency stop functionality (circuit breaker), which could be useful in case a critical issue is discovered that requires pausing the contract.

   - The contract should ensure that all external calls, especially token transfers, are handled safely to prevent reentrancy attacks. The use of checks-effects-interactions pattern is recommended.

###### 8 - StakeAtRisk.sol


**Potential Security Flaws:**

- **Centralization of Control:** The contract relies heavily on the `RankedBattle` contract, which has exclusive control over critical functions like `setNewRound`, `reclaimNRN`, and `updateAtRiskRecords`. If the `RankedBattle` contract is compromised or malicious, it could manipulate the stakes at risk.

- **Reentrancy:** The `_sweepLostStake` function calls an external contract (`_neuronInstance.transfer`). If the Neuron contract is not secure against reentrancy attacks, this could be a vulnerability. However, since Solidity 0.8.0 includes built-in overflow checks, the risk of reentrancy is mitigated unless the Neuron contract itself is flawed.

- **Lack of Input Validation:** The contract assumes that the `fighterId` and `fighterOwner` provided to the functions are valid and does not perform any checks on these inputs. Malicious or incorrect inputs could lead to unexpected behavior.

- **Authorization Checks:** The contract correctly checks that only the `RankedBattle` contract can call certain functions. However, there is no mechanism to update the `_rankedBattleAddress` in case it needs to be changed. This could be a problem if the `RankedBattle` contract needs to be upgraded or replaced.

- **Error Handling:** The contract assumes that the `transfer` function of the Neuron contract will return a boolean indicating success. If the Neuron contract does not conform to this expectation (e.g., it uses revert instead of returning false), the error handling in `reclaimNRN` and `_sweepLostStake` may not work as intended.

- **Visibility of Functions:** The `_sweepLostStake` function is private and can only be called internally, which is appropriate for its use case.

- **Contract Upgradability:** The contract does not appear to be upgradable. If any issues are found or improvements are needed, a new contract must be deployed and the state must be migrated.

###### 9 -VoltageManager.sol


**Potential Security Flaws and Considerations**:

- **Centralization Risk**: The contract is highly centralized, with significant control given to the owner and admins. If the owner's private key is compromised, the entire contract is at risk.

- **Reentrancy**: The `useVoltageBattery` function interacts with an external contract (`_gameItemsContractInstance.burn`) before updating the user's voltage. If the `GameItems` contract is malicious or has a bug, it could potentially lead to reentrancy attacks. However, since there are no further calls after the burn and no state changes before it that could be exploited, the risk is minimal in this context.

- **Integer Underflow**: In the `spendVoltage` function, the line `ownerVoltage[spender] -= voltageSpent;` could potentially underflow if `voltageSpent` is greater than `ownerVoltage[spender]`. Since Solidity 0.8.0 and above include built-in overflow/underflow checks, this would revert the transaction, but it's still a good practice to ensure that `voltageSpent` is not greater than the available voltage before attempting to subtract.

- **Access Control**: The `spendVoltage` function allows an address to spend voltage on behalf of another if they are in the `allowedVoltageSpenders` mapping. It's important to ensure that this mapping is managed securely to prevent unauthorized spending.

- **Timestamp Dependence**: The contract uses `block.timestamp` for the voltage replenish time. While this is generally safe for longer time periods like one day, it's worth noting that miners have a slight ability to manipulate block timestamps, which could potentially be exploited in more time-sensitive scenarios.

- **Magic Numbers**: The contract uses hardcoded values like `100` for voltage replenishment. It's generally a good practice to define such constants with descriptive names for better code readability and maintainability.

- **Lack of Input Validation**: There is no validation on the `newOwnerAddress` in `transferOwnership` or `adminAddress` in `adjustAdminAccess`. While not a direct security risk, it's a good practice to ensure that these addresses are not zero addresses.

- **Event Information**: The `VoltageRemaining` event is emitted after voltage is spent or replenished, which is good for transparency. However, it might be beneficial to include more information in the event, such as the amount of voltage spent or replenished.

## 5 - Architecture and Design recommendations to improve the contracts

###### AiArenaHelper.sol

The AiArenaHelper contract is designed to manage AI Arena fighters' physical attributes, including ownership transfer, attribute divisor management, and attribute generation based on DNA. 

***Here are some architecture and design recommendations to improve the contract:***

1 - **Use of Modifiers for Access Control:** Implement a modifier for owner-only functions to replace repetitive require statements.

2 - **Event Emission:** Emit events for critical state changes such as ownership transfer, attribute divisor updates, and attribute probability updates. This will help off-chain clients track changes efficiently.

3 - **Optimization of Storage:** The `attributeProbabilities` mapping uses strings as keys, which is less efficient than using integers or bytes32. Consider using an enum or a fixed-size bytes32 hash of the attribute name.

4 - **Input Validation:** Validate inputs where necessary. For example, ensure that the probabilities array provided to `addAttributeProbabilities` has valid probability values that sum up to 100 or less.

5 - **Error Messages:** Include error messages in require statements to provide more context on why a transaction failed.

6 - **Constants and Immutables:** If certain variables, such as the attributes array, are not meant to change, declare them as constant or immutable to save gas.

7 - **Function Visibility:** Functions that are not intended to be called externally should be marked as internal to save gas and reduce the contract's attack surface.

8 - **Gas Optimization:** Loops in functions like `createPhysicalAttributes` can be expensive in terms of gas. Consider strategies to reduce loop iterations or pre-compute values off-chain.

9 - **Upgradeability:** If you anticipate future changes to the contract's logic, consider implementing an upgradeable contract pattern using proxies.

10 - **Code Reusability:** If the logic for managing attributes is generic enough, consider abstracting it into a library that can be reused across different contracts.

11 - **Documentation:** Improve NatSpec comments to provide a clearer understanding of the functions, parameters, and return values.

12 - **Separation of Concerns:** Separate the concerns of ownership, attribute management, and fighter creation into different contracts or modules to adhere to the single responsibility principle.

By implementing these recommendations, the contract can become more efficient, and easier to maintain.

###### FighterFarm.sol

FighterFarm is an ERC721 token representing AI Arena Fighter NFTs. The contract includes functionality for minting, transferring, and managing these NFTs, including staking and rerolling fighters with different attributes.

***Here are some architecture and design recommendations to improve the contract:***

1 - ***Access Control:*** 

- Replace the manual owner checks with a more robust access control mechanism like `OpenZeppelin's` `Ownable` and `AccessControl` for role-based permissions.

- Use `onlyOwner` and onlyRole modifiers to protect sensitive functions.

2 - ***Gas Optimization:***

- Optimize loops and state changes to minimize gas costs, especially in functions like `claimFighters` and `redeemMintPass`.

- Use calldata instead of memory where possible to save gas.

- Consider using struct packing to optimize storage.

3 - ***Error Handling:***

- Use require with error messages for better debugging and user experience.

- Consider using custom errors to reduce gas costs and improve clarity.

4 - ***Data Validation:***

- Validate inputs, especially array lengths and enum values, to prevent out-of-bounds errors or invalid states.

- Use modifiers for repetitive input checks.

5 - ***Event Emission:*** Emit events for all state-changing operations to enable better off-chain tracking and indexing.

6 - ***State Management:***

- Avoid using arrays for storing all fighters; consider using mappings with a separate index to track the total count.

- Use enums for state variables with a limited set of possible values to improve readability and reduce errors.

- By addressing these points, the `vFighterFarm` contract can become more secure, efficient, and maintainable.

###### GameItems.sol

The GameItems is an ERC1155 token implementation with additional features tailored for the game environment. 

***Here are some architecture and design recommendations to improve the contract:***

1 - ***Access Control:*** 

- Replace the custom `owner` implementation with `OpenZeppelin's` `Ownable` contract for standardized ownership management.

- Consider using `OpenZeppelin's` `AccessControl` for more granular permissions instead of a single `isAdmin` mapping

2 - ***Events:*** Include more information in events, such as the `newOwnerAddress` in the `transferOwnership` event, to provide a complete audit trail.

3 - ***Data Structures:*** Consider using a mapping for `allGameItemAttributes` to directly access game item attributes by tokenId, which can be more efficient than iterating through an array.

4 - ***Code Reusability:*** Abstract common functionality into internal functions to avoid code duplication. For example, the replenishment logic in mint could be refactored into a separate function.

5 - ***Token URI Management:*** Consider implementing a base URI pattern to reduce the need for setting individual URIs for each token, which can be gas-intensive.

6 - ***Daily Allowance Logic:*** The daily allowance logic could be refactored to simplify the checks and reduce potential for errors.

7 - ***Decoupling:*** Decouple the `Neuron` contract interaction from the mint function to adhere to the Single Responsibility Principle.

8 - ***Immutable Variables:*** Use immutable for variables that are set once in the constructor and do not change, like `treasuryAddress`, to save gas.

By addressing these points, the contract can become more secure, efficient, and maintainable.

###### MergingPool.sol

The MergingPool contract has several aspects that can be improved in terms of architecture and design. 

***Here are some recommendations:***

1 - ***Access Control:*** Consider using `OpenZeppelin's` `Ownable` and `AccessControl` contracts for managing ownership and role-based access control. This will make the code more modular and secure by leveraging well-tested implementations.

2 - ***Events:*** Include more information in events, such as the roundId in the `PointsAdded` event and the `newOwnerAddress` in an ownership transfer event. This will improve the traceability of actions.

3 - ***Round Management:*** Introduce a struct to encapsulate round information, including the total points, winner addresses, and selection status. This will make the code cleaner and more manageable.

4 - ***Winner Selection:*** The `pickWinner` function currently assumes that the winners are fairly selected off-chain. Consider implementing a verifiable random function (VRF) or other decentralized selection mechanism to ensure fairness and transparency.

5 - ***Error Handling:*** Use custom error messages with revert to provide more informative error messages and save gas compared to require statements with string messages.

###### Neuron.sol

The Neuron contract is an ERC20 token with additional roles and functionalities such as minting, burning, and setting up airdrops. 

***Here are some architecture and design recommendations:***

1 - ***Use Role-Based Access Control (RBAC) Efficiently:*** The contract uses `AccessControl` from `OpenZeppelin`, which is good for managing roles. However, it also uses a custom `isAdmin` mapping, which seems redundant. Consider using the `AccessControl's` built-in role management for admin roles as well.

2 - ***Optimize State Variables:*** The `_ownerAddress` is used for owner-specific functions, but `AccessControl` can handle this with a default admin role. You can remove `_ownerAddress` and use `AccessControl's` `DEFAULT_ADMIN_ROLE` for a more standardized approach.

3 - ***Initial Token Distribution:*** The initial minting to the treasury and contributors is done in the constructor. Consider creating a separate function for initial distribution to make the constructor less complex and more gas-efficient.

4 - ***Event Emission:*** The `TokensMinted` event is declared but never used. Ensure that all declared events are emitted where appropriate, or remove unused events to clean up the code.

5 - ***Supply Management:*** The mint function checks if the new total supply would exceed `MAX_SUPPLY`, but it uses a strict inequality. This means that if the remaining supply is exactly equal to the amount requested for minting, the transaction will fail. Consider using `<=` to allow minting up to the max supply.

6 - ***Error Messages:*** Custom error messages are used in some require statements but not all. For consistency and to save gas, use custom error messages throughout the contract.

By addressing these points, the contract can be more secure, efficient, and maintainable.

###### RankedBattle.sol

The RankedBattle contract is designed to handle staking, battle records, and reward distribution for a game. 

***Here are some architecture and design recommendations to improve the contract:***

1 - ***Access Control:*** Instead of using a single `_ownerAddress` for privileged actions, consider using `OpenZeppelin's` `AccessControl` for more granular permissions with different roles.

2 - ***Access Control:*** Instead of using a single `_ownerAddress` for privileged actions, consider using `OpenZeppelin's` `AccessControl` for more granular permissions with different roles.

3 - ***Error Handling:*** Use revert with custom error messages instead of require to save gas and improve readability.

4 - ***Event Indexing:*** Index important event parameters to make it easier to filter events.

5 - ***State Variable Visibility:*** Explicitly declare the visibility of state variables (e.g., private, internal) for clarity and security.

6 - ***Modularization:*** Break down the contract into smaller, more focused contracts to improve readability and maintainability. For example, separate the staking logic from the battle record management.

7 - ***Pull Over Push for Rewards:*** Instead of pushing rewards to users (which can fail if the user's contract has a fallback function), let users pull their rewards. This is already implemented with the `claimNRN` function, but ensure all reward distributions follow this pattern.

8 - ***Input Validation:*** Validate inputs thoroughly, especially for critical functions like `stakeNRN` and `unstakeNRN`, to prevent unexpected behavior.

9 - ***Time Manipulation:*** Be aware of potential risks with relying on `block.timestamp` for critical game mechanics, as miners can manipulate it to a certain degree.

10 - ***Circuit Breaker:*** Implement a circuit breaker or pause functionality to disable critical contract functions in case of an emergency or discovery of a critical bug.

By addressing these points, the RankedBattle contract can become more robust, secure, and maintainable.

###### StakeAtRisk.sol

The StakeAtRisk contract is designed to manage the NRNs that are at risk of being lost during a round of competition. 

***Here are some architecture and design recommendations to improve the contract:***

1 - ***Access Control:*** Consider using `OpenZeppelin's` `Ownable` or `AccessControl` for more granular permissions instead of only allowing the `_rankedBattleAddress` to call certain functions.

2 - ***Round Management:*** 

- Implement checks to prevent setting a new round if the previous round has not been properly concluded or if there are still unresolved stakes.

- Consider adding a function to manually or automatically conclude a round, ensuring all stakes are accounted for before moving to the next round.

3 - ***Error Handling:***

- In the `reclaimNRN` function, if the transfer fails, the state is still updated. Use the `Checks-Effects-Interactions` pattern to prevent this.

- In the `setNewRound` function, if the sweep fails, the round is still updated. Consider reverting the transaction if the sweep is not successful.

4 - Event Emission: Emit events for critical state changes, such as when a new round is set or when stakes are swept to the treasury.

By implementing these recommendations, the contract can become more secure, efficient, and maintainable.

VoltageManager.sol

The VoltageManager contract is designed to manage a voltage system for the game, with functionalities to transfer ownership, adjust admin access, manage allowed voltage spenders, use voltage batteries, and spend voltage. 

***Here are some architecture and design recommendations:***

1 - ***Use of OpenZeppelin's Ownable Contract:*** Instead of manually implementing ownership functionality, consider inheriting from `OpenZeppelin's` `Ownable` contract, which provides a well-tested and secure implementation of ownership with features like `transferOwnership` and `onlyOwner` modifiers.

2 - ***Role-Based Access Control:*** Instead of a binary admin flag, consider using `OpenZeppelin's` `AccessControl` for more granular role-based permissions. This allows for different roles with specific permissions, making the system more flexible and secure.

3 - ***Reentrancy Guard:*** Functions that interact with external contracts, such as `useVoltageBattery`, should be protected against reentrancy attacks. Use `OpenZeppelin's` `ReentrancyGuard` or check-effects-interactions pattern to mitigate such risks.

4 - ***Underflow Check:*** The `spendVoltage` function should check for underflow when subtracting voltage. `Solidity 0.8.x` has built-in overflow/underflow checks, but it's good practice to ensure that the `voltageSpent` is not greater than the `ownerVoltage[spender]` before performing the subtraction.

5 - ***Event Information:*** The `VoltageRemaining` event could include more information, such as the action taken (e.g., spent or replenished) and the timestamp, to provide a more detailed history of voltage changes.

6 - ***Time Manipulation:*** The use of `block.timestamp` can be manipulated by miners to a certain degree. If the precision of the replenish time is critical, consider using an external time oracle or a block number-based approach.

7 - ***Input Validation:*** Validate inputs where necessary. For example, in `transferOwnership` and `adjustAdminAccess`, ensure that the `newOwnerAddress` and `adminAddress` are not zero addresses.

By implementing these recommendations, the contract can be more secure, efficient, and easier to maintain.

















### Time spent:
60 hours