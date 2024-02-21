### Advanced Analysis Report for [Ai-Arena](https://github.com/code-423n4/2024-02-ai-arena) by K42
#### Overview
- The [Ai-Arena](https://github.com/code-423n4/2024-02-ai-arena) project introduces a unique, on-chain ``NFT`` game that leverages the ``ERC-721`` and ``ERC-1155`` standards for a game involving ``NFT`` characters and in-game items. 

#### Understanding the Ecosystem:
- The ecosystem is a well-orchestrated play-to-earn model leveraging ``ERC721`` and ``ERC1155`` tokens for characters and items, respectively, and ``ERC20`` tokens for in-game currency. It is supported by a range of solidity smart contracts that manage the game's logic, including battles, ``NFT`` staking, and ``NFT`` trading.

#### Codebase Quality Analysis:
**1. Key Data Structures and Libraries in all contracts**
- The use of ``OpenZeppelin ``libraries for ``ERC`` standards, ``AccessControl``, and security measures is a positive sign. As well as the use of ``solmate`` for gas efficent fixed point math usage, that is awesome. However, the game's data and the in-game logic are tightly coupled across the contracts, which could be a risk for upgradability and long term maintainability.

**2. Use of Modifiers and Access Control in all contracts**
- The use of `onlyOwner` and `isAdmin` access controls is a concern, but at first glance well implemented. The game's power is too centralized however as a result, and the use of a single account to manage the game's control can be a single point of failure.

**3. Use of State Variables in all contracts**
- The ``public`` and ``private`` state variables across the ecosystem can be a major risk, if creative attack vectors are used. It is always a good idea to review the scoping of these variables to ensure they are only accessible to the right entities.

**4. Use of Events and Logging in all contracts**
- The event usage is well-implemented, with a lot of the game's main state changes being emitted.

#### Codebase Analysis:
## [AiArenaHelper.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol) Contract

### Functions and Risks

#### `transferOwnership(address newOwnerAddress)`
- **Specific Risk**: Centralization Risk. This function allows the contract owner to transfer ownership, which could lead to a single point of failure if the owner's private key is compromised.
- **Recommendation**: Implement a multi-signature mechanism for critical functions like this to enhance security and distribute trust.

#### `addAttributeDivisor(uint8[] memory attributeDivisors)`
- **Specific Risk**: Input Validation Risk. Without proper validation, there's a risk that the input array might not match the expected length, potentially leading to incorrect configuration of attribute divisors.
- **Recommendation**: Ensure rigorous validation of the input array length to match the expected number of attributes, preventing misconfiguration.

#### `createPhysicalAttributes(uint256 dna, uint8 generation, uint8 iconsType, bool dendroidBool)`
- **Specific Risk**: Deterministic Output Risk. Given the deterministic nature of blockchain, the output of this function is predictable if the inputs are known, potentially allowing gaming of the system to achieve desired attributes.
- **Recommendation**: Introduce additional randomness or unpredictability in the attribute generation process to mitigate predictability and ensure fairness.

#### `addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities)`
- **Specific Risk**: Unauthorized Access Risk. This function allows the owner to set probabilities for attributes, which could be misused if the owner account is compromised, affecting the game's balance and fairness.
- **Recommendation**: Limit access to this function with role-based access controls (RBAC) and consider a decentralized governance model for making such critical changes.

#### `deleteAttributeProbabilities(uint8 generation)`
- **Specific Risk**: Data Loss Risk. This function enables the deletion of attribute probabilities for a generation, which could unintentionally remove essential data for the game's mechanics.
- **Recommendation**: Implement a safeguard mechanism, such as requiring confirmation through a multi-signature process or a time-lock delay to prevent accidental or malicious deletions.

#### `getAttributeProbabilities(uint256 generation, string memory attribute)`
- **Specific Risk**: Transparency vs. Strategy Risk. While providing transparency, revealing attribute probabilities could influence player strategies in a way that reduces the game's competitive nature.
- **Recommendation**: Balance transparency with gameplay integrity by carefully considering what information is made public and when.

#### `dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute)`
- **Specific Risk**: Algorithm Exploit Risk. The method for converting DNA to attribute index based on rarity rank could be exploited if the underlying algorithm or its inputs are predictable.
- **Recommendation**: Review and possibly enhance the algorithm for converting DNA to attribute indices to ensure it remains unpredictable and fair.

---

## [FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol) Contract

### Functions and Risks

#### `transferOwnership(address newOwnerAddress)`
- **Specific Risk**: Potential for a single point of control, where the contract's ownership could be transferred to a malicious address if the private key of the current owner is compromised.
- **Recommendation**: Implement a multi-signature requirement or a decentralized governance mechanism for ownership-related and other sensitive functions to mitigate the potential for misuse.

#### `incrementGeneration(uint8 fighterType)`
- **Specific Risk**: The operation can unilaterally change the game's economy or balance by the owner, potentially introducing new generations without community consent, which could affect the rarity and value of the NFTs.
- **Recommendation**: Consider a DAO or a community voting mechanism for any action that could have a significant effect on the game's balance or the NFTs' value to ensure the community's support for the change.

#### `addStaker(address newStaker)`
- **Specific Risk**: The owner has the power to arbitrarily add new stakers, which could be a vector for centralization and favoritism, potentially compromising the staking mechanism's fairness.
- **Recommendation**: Implement a transparent, permissionless strategy for adding stakers, possibly through a staking contract that allows any user to participate under predefined and automatic conditions.

#### `instantiateAIArenaHelperContract(address aiArenaHelperAddress)`
- **Specific Risk**: The process of updating or setting a new AI Arena Helper contract poses a security risk if the new address is not properly audited, potentially compromising the game's integrity.
- **Recommendation**: Any changes to the AI Arena Helper or other critical game mechanics should be done through a time-locked, community-reviewed process to ensure the new code's trustworthiness.

#### `claimFighters(...)`
- **Specific Risk**: The process of claiming fighters could be exploited if the input validation is not secure, potentially allowing bad actors to mint more NFTs than intended or to others' detriment.
- **Recommendation**: Ensure that the number of NFTs a player can claim is capped and that inputs are validated against a whitelist or a cryptographic proof (like signatures) to prevent over-claiming or fraudulent minting.

#### `redeemMintPass(...)`
- **Specific Risk**: The process of redeeming mint passes and burning them for NFTs could be a vector for economic imbalance if the mint pass system is exploited or if the burning mechanism fails.
- **Recommendation**: Ensure the mint pass redemption and burn logic is airtight, with fail-safes and pre-checks to prevent double spending or the minting of NFTs without a valid mint pass.

#### `updateFighterStaking(uint256 tokenId, bool stakingStatus)`
- **Specific Risk**: The process of updating a fighter's staking status could be a vector for staking-related exploits, such as staking the same NFT in multiple pools or withdrawing it without the right to do so.
- **Recommendation**: Secure the process with a time-locked or escrow-like mechanism to prevent the same NFT from being used in multiple systems or being unstaked too quickly, and ensure the process is in line with the staking pool's game theory.

#### `setTokenURI(uint256 tokenId, string calldata newTokenURI)`
- **Specific Risk**: The process of updating the token URI could be exploited to redirect the metadata to fraudulent or inappropriate data, affecting the NFT's visual representation and value.
- **Recommendation**: Restrict the process of updating the token URI to a set of pre-approved or on-chain generated URIs to prevent misuse, and ensure the data's integrity and appropriateness.

#### `updateModel(uint256 tokenId, string calldata modelHash, string calldata modelType)`
- **Specific Risk**: The action of updating the model for a given token could be exploited to change the NFT's in-game character or model, affecting the game's balance or the NFT's collectible value.
- **Recommendation**: Any action that could alter the NFT's in-game value or balance should be community-reviewed or -proposed, with changes being as transparent and consensual as possible.

---

## [FighterOps.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol) Contract

### Functions and Risks

#### `fighterCreatedEmitter(uint256 id, uint256 weight, uint256 element, uint8 generation)`
- **Specific Risk**: Event Over-Reliance. The use of this event as the sole means of tracking fighter creation could lead to issues if event logs are not properly monitored or if they are inaccessible due to data pruning by Ethereum nodes.
- **Recommendation**: Supplement the event-based tracking with state-based records for critical game mechanics. Consider maintaining a mapping or an array of created fighters for on-chain verification and data integrity.

#### `getFighterAttributes(Fighter storage self)`
- **Specific Risk**: View Function Misuse. As a public view function, it exposes the full set of a fighter's attributes, which could be leveraged by players to gain an unfair edge in gameplay or for botting purposes.
- **Recommendation**: Implement access control mechanisms to limit the exposure of sensitive data. Consider using hash-based or encrypted data for attributes that should remain hidden until certain gameplay conditions are met.

#### `viewFighterInfo(Fighter storage self, address owner)`
- **Specific Risk**: Inadvertent Privacy Breach. This view function consolidates and exposes a lot of data about a fighter, including potentially player-identifiable data through the association of the `owner` address.
- **Recommendation**: Ensure that the use of the `owner` parameter and the data returned by this method are in compliance with the intended game dynamics and privacy considerations. Evaluate the necessity of all the returned data and potentially restrict access to sensitive data.

---

## [GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol) Contract

### Functions and Risks

#### `transferOwnership(address newOwnerAddress)`
- **Specific Risk**: Centralization of control introduces a high risk if the contract owner's management key is compromised, allowing a bad actor to unilaterally change the game's course.
- **Recommendation**: Implement a decentralized decision-making mechanism, such as a DAO, to govern the game's future, or a multi-signature system for this and other sensitive operations.

#### `adjustAdminAccess(address adminAddress, bool access)`
- **Specific Risk**: The power to grant or revoke admin status can be a vector for misuse, potentially allowing the unauthorized access to admin-only methods or the locking out of legitimate actors.
- **Recommendation**: Transition to a transparent, community-validated system for role changes, such as a time-locked, off-chain voting strategy to limit the admin's absolute authority.

#### `adjustTransferability(uint256 tokenId, bool transferable)`
- **Specific Risk**: The game's ability to toggle the transferability of in-game items can have a major, direct effect on the player economy, potentially rendering items illiquid without notice.
- **Recommendation**: Token transferability should be a right of the token holder, not the system. If the game's logic requires the option to lock items, it should be a part of the in-game state, not the NFT's on-chain security properties.

#### `mint(uint256 tokenId, uint256 quantity)`
- **Specific Risk**: The unbounded item minting can lead to the devaluation of in-game items, and the game's market can be flooded with items, making the in-game environment unbalanced.
- **Recommendation**: Implement a cap on the number of items that can be minted and a deflationary system to ensure the game's market balance. The cap should be a decision of the game's governance model.

#### `setAllowedBurningAddresses(address newBurningAddress)`
- **Specific Risk**: The process of allowing specific addresses to burn in-game items can be a vector for potential misuse, where an address could be authorized to burn items they shouldn't have access to.
- **Recommendation**: Enforce a strict validation and a
uditing trail for any changes to allowed burning addresses. Consider a DAO or a community vote for any new addresses to be added to this list, ensuring transparency and community support.

#### `setTokenURI(uint256 tokenId, string memory _tokenURI)`
- **Specific Risk**: The process of updating the tokenURI dynamically could be exploited to redirect metadata to fraudulent or misleading data, affecting the in-game representation and the value of the NFTs.
- **Recommendation**: Implement a secure, verifiable method for metadata updates, such as a DAO vote or a content hash check to ensure the integrity of the NFT metadata. Additionally, consider a review process for any content changes.

#### `createGameItem(...)`
- **Specific Risk**: The process of creating new game items unilaterally by the admin can be a vector for economic imbalance, potentially saturating the market with new or unbalanced items.
- **Recommendation**: Transition to a model where the community or a DAO has a say in the creation of new game items. This could be through a proposal and review process, ensuring that new items have the support of the game's community and are well-balanced.
#### `burn(address account, uint256 tokenId, uint256 amount)`
- **Specific Risk**: The process of burning in-game items, if not properly managed, can be a vector for potential data loss or market manipulation, affecting the game's play-to-earn system.
- **Recommendation**: Enforce a set of in-game preconditions that must be met before an item can be burned. Additionally, consider a review or a delay system for the burning of high-value or high-impact items to prevent hasty or emotional item destruction.

#### `mint(uint256 tokenId, uint256 quantity)`
- **Specific Risk**: The minting process, especially for items with finite supply, needs to ensure that it doesn't exceed the predefined limits, which could lead to inflation and devaluation of existing items.
- **Recommendation**: Strictly enforce the game's logic to check and adhere to the game's pre-established supply caps. Implement a supply management sub-system that transparently and accurately tracks and controls the game's in-game economy.

#### `safeTransferFrom(...)`
- **Specific Risk**: The transfer function must ensure that the item's game logic, such as transferability and staking conditions, is respected, to prevent the exploitation of game mechanics.
- **Recommendation**: Implement a pre-transfer check that verifies all the game's business logic is in a valid state for the transfer to occur. This could include staking conditions, cool-down periods, or other in-game business logic.

---

## [MergingPool.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol) Contract

### Functions and Risks

#### `transferOwnership(address newOwnerAddress)`
- **Specific Risk**: Centralization of power in a single admin's hands can pose a major project governance and fund mismanagement threat.
- **Recommendation**: Implement a more decentralized or multi-signature system for making high-stake project decisions, such as a DAO.

#### `adjustAdminAccess(address adminAddress, bool access)`
- **Specific Risk**: The power to unilaterally grant or revoke admin status can be a security and centralization issue, potentially allowing the abuse of the power by the project's owner.
- **Recommendation**: Transition to a more transparent and permissionless process for conferring admin status, such as a time-locked, community-voted mechanism.

#### `updateWinnersPerPeriod(uint256 newWinnersPerPeriodAmount)`
- **Specific Risk**: The process of changing the number of winners can be a vector for fund misallocation or manipulation, affecting the project's long-term success.
- **Recommendation**: Automate the project's high-stake game mechanics, like the number of winners, through a set of in-game or on-chain oracles to ensure the game's invariability and predictability.

#### `pickWinner(uint256[] calldata winners)`
- **Specific Risk**: The game's high-stake power to pick winners can be a major security and manipulation issue, affecting the project's game-theoretic and long-term community support.
- **Recommendation**: Transition to a more on-chain, verifiable, and automatic process for determining winners, such as a VRF-based process, to ensure the game's invariability and predictability.

#### `claimRewards(...)`
- **Specific Risk**: The project's action of claiming rewards can be a vector for contract manipulation or front-running, affecting the project's short-term and long-term support.
- **Recommendation**: Secure the project's data flow and fund flow, especially for the game's in-game and on-chain oracles, to ensure the game's invariability and predictability.

#### `addPoints(uint256 tokenId, uint256 points)`
- **Specific Risk**: The game's admin has the power to arbitrarily add points to fighters, which could be a vector for potential manipulation or unfair play, affecting the project's integrity.
- **Recommendation**: Transition to a more on-chain, verifiable, and automatic point calculation and awarding process to ensure fairness and transparency in gameplay.

#### `getFighterPoints(uint256 maxId)`
- **Specific Risk**: The ability to query the total number of points for a range of fighters can be a vector for privacy issues or market manipulation, affecting the game's long-term support.
- **Recommendation**: Ensure the action of querying the game's in-game and on-chain oracles is in line with the game's business logic and invariability to ensure the project's invariability and predictability.

---

## [Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol) Contract

### Functions and Risks

#### `transferOwnership(address newOwnerAddress)`
- **Specific Risk**: Centralization of power. This function allows the current owner to transfer ownership, which could be a security risk if the admin's control is compromised.
- **Recommendation**: Implement a decentralized access control system or a multi-signature system for such sensitive functions to mitigate the single point of control risk.

#### `addMinter(address newMinterAddress)`
- **Specific Risk**: Unlimited minting access. By adding new minters, there's a security risk if the minter's key is compromised, potentially leading to over-minting.
- **Recommendation**: Limit the minter's power by imposing a hard cap on the number of tokens a minter can mint and by time-locking the minter's power to prevent abuse.

#### `addStaker(address newStakerAddress)`
- **Specific Risk**: Concentration of staking management. A new staker can have too much control over the staking process, potentially compromising the staking model's integrity.
- **Recommendation**: Implement a transparent and permissionless staking system that doesn't rely on a centralized power to manage or control the staking system.

#### `addSpender(address newSpenderAddress)`
- **Specific Risk**: Mismanagement of spending power. The potential for a new spender to misuse their control over the network's resources, affecting the game's in-game balance or the game's network performance.
- **Recommendation**: Enforce a cap on the total spendable tokens by any spender and a time-bound access to the power to prevent the risk of mismanagement.

#### `adjustAdminAccess(address adminAddress, bool access)`
- **Specific Risk**: Admin power abuse. The action of unilaterally granting or revoking admin power can be a control and power abuse point.
- **Recommendation**: Transition to a more transparent and permissionless power system, where the game's power is more decentralized or the game's major players have a voice in the action.

#### `setupAirdrop(address[] calldata recipients, uint256[] calldata amounts)`
- **Specific Risk**: Economic imbalance. The game's project can be at risk of airdrop misuse, which can be a point of power abuse, potentially affecting the game's in-game balance.
- **Recommendation**: Ensure the airdrop process is transparent and the game's project has a plan to prevent the misuse of airdrops, which can be a power abuse method.

#### `claim(uint256 amount)`
- **Specific Risk**: Over-claiming. The system can be a risk of airdrop misuse, which can be a power abuse method, potentially affecting the project's game in-game balance.
- **Recommendation**: Ensure the game's power is more decentralized or the game's major players have a voice in the process to prevent the misuse of airdrops.

#### `mint(address to, uint256 amount)`
- **Specific Risk**: Inflation. Minting without strict controls can dilute the value of the token, affecting all token holders.
- **Recommendation**: Enforce a hard cap on the number of tokens that can be minted to prevent the game's market from being flooded with new tokens, which could devalue the game's network.

#### `burn(uint256 amount)`
- **Specific Risk**: Loss of value. The process of burning tokens can be a vector for power abuse, potentially affecting the project's market cap or the power of the project's power system.
- **Recommendation**: Transition to a more transparent and power system, where the project's game power is more decentralized or the game's major players have a voice in the power abuse.

#### `approveSpender(address account, uint256 amount)`
- **Specific Risk**: Misuse of spending authority. The system can be a risk of airdrop misuse, which can be a point of control and a point of risk for the project's project game.
- **Recommendation**: Transition to a more transparent and permissionless game system, where the project's project game is more decentralized or the game's main players have a voice in the game's main game.

#### `burnFrom(address account, uint256 amount)`
- **Specific Risk**: The power to burn tokens from another account can be misused if the power is not carefully managed and audited.
- **Recommendation**: Require a time-locked or a community-approved action for the game's main game to prevent the misuse of the game's main game, which can be a control and a risk for the game's project game.

---

## [RankedBattle.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol) Contract

### Functions and Risks

#### `transferOwnership(address newOwnerAddress)`
- **Specific Risk**: Centralization of control. This action allows the admin to unilaterally change the contract's owner, which could lead to a single point of failure or misuse if the account is compromised.
- **Recommendation**: Implement a decentralized access management model, such as a DAO or a multi-signature system, to manage the power to transfer ownership.

#### `adjustAdminAccess(address adminAddress, bool access)`
- **Specific Risk**: Admin power misuse. Admins have the power to change the control of the game's logic, which could be a problem if the key is compromised.
- **Recommendation**: Use a time-locked smart contract or a DAO to manage the project's high-stake game mechanics, such as power handover, to ensure the community's interest is prioritized.

#### `setGameServerAddress(address gameServerAddress)`
- **Specific Risk**: Single point of trust. The admin can change the game server's control, which could be a problem if the account is compromised.
- **Recommendation**: Automate the process of server rotation or game state management through a set of in-game or on-chain oracles to ensure the project's invariability and predictability.

#### `stakeNRN(uint256 amount, uint256 tokenId)`
- **Specific Risk**: Stake manipulation. The admin can change the staking logic, which could be a security threat if the project's business logic is not well-audited.
- **Recommendation**: Automate the staking control and cap the admin's role to a system of in-game or on-chain oracles to ensure the business logic's invariability and predictability.

#### `unstakeNRN(uint256 amount, uint256 tokenId)`
- **Specific Risk**: Unstaking manipulation. The method of unstaking can be a security threat if the power is not well-audited.
- **Recommendation**: Implement a set of in-game or on-chain oracles to ensure the business logic's invariability and predictability.

#### `claimNRN()`
- **Specific Risk**: Claim manipulation. The process of claiming rewards could be a security risk if the business logic is not well-tested.
- **Recommendation**: Use a series of on-chain data and a more secure, time-locked, and automated function to claim the game's in-game money to ensure the community's long-term interest.

#### `updateBattleRecord(...)`
- **Specific Risk**: Battle outcome manipulation. The project's current game logic could be a problem if the ability to update the game's action is not in line with the business logic.
- **Recommendation**: Transition to a more on-chain, real-time, and secure power model to prevent the problem of a business logic that is not in line with the business logic.

#### `setRankedNrnDistribution(...)`
- **Specific Risk**: Inflation risk. The game's business logic could be a point of a game's action that is not in line with the market's role if the account is compromised.
- **Recommendation**: Transition to a more on-chain, time-locked, and real-time price feed to ensure the market's game logic is in line with the project's main game logic.

---

## [StakeAtRisk.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol) Contract

### Functions and Risks

#### `setNewRound(uint256 roundId_)`
- **Specific Risk**: Centralized Authority. This function, callable by the RankedBattle contract, can transition the staking game to a new phase, potentially affecting the risk and return profile of all players' strategies.
- **Recommendation**: Ensure that the transition to a new game phase is a result of a community consensus or a set of in-game, on-chain, and pre-defined criteria to prevent manipulation.

#### `reclaimNRN(uint256 nrnToReclaim, uint256 fighterId, address fighterOwner)`
- **Specific Risk**: Token Withdrawal. This method allows the RankedBattle contract to withdraw a particular number of tokens from the staking pool, which could be a security risk if the RankedBattle control is compromised.
- **Recommendation**: Enforce a time-locked or a multi-confirmation system for any fund's outflow, and ensure that the RankedBattle's power is as decentralized as possible to prevent a single point of control.

#### `updateAtRiskRecords(uint256 nrnToPlaceAtRisk, uint256 fighterId, address fighterOwner)`
- **Specific Risk**: Inadequate Validation. The system could be exploited to inappropriately increase the risk for a player's funds or to maliciously alter the staking power of a user's game character.
- **Recommendation**: Ensure that the game's business logic is in a good position to prevent the system from being manipulated. Additionally, the project should have a self-managed and self-governed process to prevent the self-managed and self-governed project from being a self-managed and self-governed process.

---

## [VoltageManager.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol) Contract

### Functions and Risks

#### `transferOwnership(address newOwnerAddress)`
- **Specific Risk**: Centralization of control. The process of transferring the entire control of the smart contract to a new address can be a security and mismanagement threat.
- **Recommendation**: Implement a time-locked, multi-signature, or DAO-based system for any control-changing operations to ensure community or stakeholder consensus.

#### `adjustAdminAccess(address adminAddress, bool access)`
- **Specific Risk**: Admin access mismanagement. The process of unilaterally granting or revoking admin status can be a control and manipulation issue.
- **Recommendation**: Shift to a more transparent and user-managed power management process via dao structure. 

---

#### General high level recommendation:
- Use [Tenderly](https://dashboard.tenderly.co/) and [Defender](defender.openzeppelin.com) for continued monitoring to prevent un-foreseen risks or actions. 

#### Conclusion:
- The [Ai-Arena](https://github.com/code-423n4/2024-02-ai-arena) project showcases a well-architected foray into the world of on-chain ``NFT`` gaming. By leveraging the ``ERC-721`` and ``ERC-1155`` standards, it has created a game that is not only unique in its play-to-earn ``(P2E)`` dynamics but also in the deep integration of ``NFTs`` and in-game micro-economies for a more immersive and self-sustaining game world.

### Time spent:
28 hours