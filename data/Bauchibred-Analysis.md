# Analysis Report for AI Arena

## Table of Contents

- [Approach](#approach)
- [Brief Overview](#brief-overview)
- [Scope and Architecture Overview](#scope-and-architecture-overview)
- [Centralization Risks](#centralization-risks)
- [Systemic Risks](#systemic-risks)
- [Recommendations](#recommendations)
- [Security Researcher Logistics](#security-researcher-logistics)
- [Conclusion](#conclusion)
- [Resources](#resources)

## Approach

Started with a comprehensive analysis of the provided docs to understand protocol functionality and key points, ambiguities were discussed with the sponsors(deva), and possible risk areas were outlined.

Followed by a manual review of each contract in scope, testing function behavior, protocol logic against expectations, and working out potential attack vectors. Vulnerabilities related to dependencies and inheritances, were also assessed. Comparisons with similar protocols was also performed to identify recurring issues and evaluate fix effectiveness.

Finally, issues identified during the security review were compiled into a comprehensive audit report.

## Brief Overview

AI Arena introduces a player-versus-player combat game set in a realm where human-trained AI fighters clash for supremacy. In the Web3 adaptation of AI Arena, these fighters are represented as NFTs through the` FighterFarm.sol` contract, where each NFT fighter is characterized by:

- Physical Attributes: Shape the visual aesthetics of the fighter.
- Generation: Also influences the fighter's visual traits.
- Weight: Impacts the combat capabilities.
- Element: Bestows unique special powers.
- Fighter Type: Distinguishes between a standard Champion or a Dendroid.
- Model Data: Includes the model type and its corresponding hash.

## Scope and Architecture Overview

### AIArenaHelper.sol

This contract helps generate and manage the AI Arena fighters physical attributes, having it's constructor initializing the contract with the attribute probabilities for gen 0.
**State Variables:**

- `attributes`: List of fighter attributes (head, eyes, mouth, body, hands, feet).
- `defaultAttributeDivisor`: Default DNA divisors for each attribute.
- `_ownerAddress`: Address of the contract owner.

**Mappings:**

- `attributeProbabilities`: Tracks fighter generation to attribute probabilities.
- `attributeToDnaDivisor`: Maps attribute to its DNA divisor.

**Constructor:**

- Initializes the contract with attribute probabilities for generation 0.

**External Functions:**

- `transferOwnership`: Transfers ownership of the contract.
- `addAttributeDivisor`: Adds attribute divisors for attributes.
- `createPhysicalAttributes`: Creates fighter physical attributes based on DNA.

**Public Functions:**

- `addAttributeProbabilities`: Adds attribute probabilities for a generation.
- `deleteAttributeProbabilities`: Deletes attribute probabilities for a generation.
- `getAttributeProbabilities`: Gets attribute probabilities for a generation and attribute.
- `dnaToIndex`: Converts DNA and rarity rank into an attribute probability index.

```css
+---------------------------------------+
| AiArenaHelper - 95 sLOC |
+---------------------------------------+
| - attributes: string[] |
| - defaultAttributeDivisor: uint8[] |
| - _ownerAddress: address |
| - attributeProbabilities: mapping |
| - attributeToDnaDivisor: mapping |
+---------------------------------------+
| + constructor(probabilities) |
| + transferOwnership(newOwner) |
| + addAttributeDivisor(divisors) |
| + createPhysicalAttributes(dna, |
| generation, iconsType, |
| dendroidBool) |
| + addAttributeProbabilities( |
| generation, probabilities) |
| + deleteAttributeProbabilities( |
| generation) |
| + getAttributeProbabilities( |
| generation, attribute) |
| + dnaToIndex(generation, rarityRank, |
| attribute) |
+---------------------------------------+
```

### FighterFarm.sol

**Purpose:**

- Manages AI Arena Fighter NFTs (ERC721 tokens).
- Also handles creation, ownership, transfers, redemption of mint passes, merging pool interactions, fighter staking, rerolling, model updates, and metadata retrieval.

**Key Roles and Permissions:**

- **Owner:** Privileged address with full control over contract functions (e.g., ownership transfer, generation increment).
- **Delegated Address:** Authorized to set token URIs and sign fighter claim messages.
- **Stakers:** Addresses allowed to stake fighters on behalf of users.

**NFT Creation and Management:**

- **Minting:**
  - Occurs through claiming (with signature verification), mint pass redemption, or merging pool minting.
- **Rerolling:**
  - Regenerates a fighter's traits with a Neuron token (NRN) cost, limited by a maximum reroll count.
- **Model Updates:**
  - Allows owners to update the ML model associated with their fighters.
- **Staking:**
  - Controls a fighter's ability to be transferred.

**Interactions with Other Contracts:**

- **AIArenaHelper:**
  - Assists with physical attribute generation.
- **AAMintPass:**
  - Interacts with mint pass redemption.
- **Neuron:**
  - Handles NRN token transfers for rerolling.
- **Merging Pool:**
  - Facilitates minting from the merging pool.

**Additional Features:**

- **URI Storage:**
  - Stores contract and token metadata on IPFS.
- **Fighter Information Retrieval:**
  - Provides a function to access comprehensive fighter data.

```css

+---------------------------------------+
| FighterFarm - 327 sLOC |
+---------------------------------------+
| - MAX_FIGHTERS_ALLOWED: uint8 |
| - maxRerollsAllowed: uint8[2] |
| - rerollCost: uint256 |
| - generation: uint8[2] |
| - totalNumTrained: uint32 |
| - treasuryAddress: address |
| - _ownerAddress: address |
| - _delegatedAddress: address |
| - _mergingPoolAddress: address |
| - _aiArenaHelperInstance: AiArenaHelper|
| - _mintpassInstance: AAMintPass |
| - _neuronInstance: Neuron |
| - fighters: FighterOps.Fighter[] |
| - fighterStaked: mapping |
| - numRerolls: mapping |
| - hasStakerRole: mapping |
| - numElements: mapping |
| - nftsClaimed: mapping |
| - numTrained: mapping |
| - _tokenURIs: mapping |
+---------------------------------------+
| + constructor(owner, delegated, treasury)|
| + transferOwnership(newOwner) |
| + incrementGeneration(fighterType) |
| + addStaker(newStaker) |
| + instantiateAIArenaHelperContract(helper)|
| + instantiateMintpassContract(mintpass)|
| + instantiateNeuronContract(neuron) |
| + setMergingPoolAddress(mergingPool) |
| + setTokenURI(tokenId, newTokenURI) |
| + claimFighters(numToMint, signature, modelHashes, modelTypes)|
| + redeemMintPass(mintpassIdsToBurn, fighterTypes, iconsTypes, mintPassDnas, modelHashes, modelTypes)|
| + updateFighterStaking(tokenId, stakingStatus)|
| + updateModel(tokenId, modelHash, modelType)|
| + doesTokenExist(tokenId) |
| + mintFromMergingPool(to, modelHash, modelType, customAttributes)|
| + transferFrom(from, to, tokenId) |
| + safeTransferFrom(from, to, tokenId) |
| + reRoll(tokenId, fighterType) |
| + contractURI() |
| + tokenURI(tokenId) |
| + supportsInterface(interfaceId) |
| + getAllFighterInfo(tokenId) |
+---------------------------------------+
```

### GameItems.sol

**Purpose:**

This contract manages a collection of in-game items used in AI Arena, allowing users to:

- Buy items with the Neuron (NRN) token.
- Mint items with finite or infinite supply.
- Burn items (only authorized addresses).
- Transfer items (if allowed), with admin control over transferability.
- Manage item attributes (name, price, daily allowance, etc.) by admins.

**Key Features:**

- **ERC1155 compliant:** Manages multiple item types with a single contract.
- **Daily allowance:** Limits the number of items a user can buy per day.
- **Transferable/locked:** Admins can control if items can be traded.
- **Admin roles:** Defines addresses with special permissions.
- **Metadata storage:** Stores item data on IPFS.

**Contract Structure:**

- **State Variables:** Store contract data like item attributes, treasury address, owner address, etc.
- **Mappings:** Track user allowances, replenish times, burning/admin permissions, and token URIs.
- **Events:** Emit signals for item purchases, locking/unlocking, and more.
- **Functions:**
  - **External:** Public functions for buying, transferring, burning, and managing items (admins only).
  - **Public:** View information about items and allowances.
  - **Private:** Internal functions for managing allowances and metadata.

**Additional Notes:**

- The contract interacts with the `Neuron` contract for NRN token transfers.
- Only authorized addresses can burn items.
- Admins have full control over item creation, attributes, and transferability.
- The contract uses an ERC1155 base for efficient management of multiple item types.

```css

+---------------------------------------+
| GameItems - 163 sLOC |
+---------------------------------------+
| - name: string |
| - symbol: string |
| - allGameItemAttributes: GameItemAttributes[] |
| - treasuryAddress: address |
| - _ownerAddress: address |
| - _itemCount: uint256 |
| - _neuronInstance: Neuron |
| - allowanceRemaining: mapping |
| - dailyAllowanceReplenishTime: mapping|
| - allowedBurningAddresses: mapping |
| - isAdmin: mapping |
| - _tokenURIs: mapping |
+---------------------------------------+
| + constructor(owner, treasury) |
| + transferOwnership(newOwner) |
| + adjustAdminAccess(admin, access) |
| + adjustTransferability(tokenId, transferable)|
| + instantiateNeuronContract(nrnAddress)|
| + mint(tokenId, quantity) |
| + setAllowedBurningAddresses(newBurningAddress)|
| + setTokenURI(tokenId, _tokenURI) |
| + createGameItem(name, tokenURI, finiteSupply, transferable, itemsRemaining, itemPrice, dailyAllowance) |
| + burn(account, tokenId, amount) |
| + contractURI() |
| + uri(tokenId) |
| + getAllowanceRemaining(owner, tokenId)|
| + remainingSupply(tokenId) |
| + uniqueTokensOutstanding() |
| + safeTransferFrom(from, to, tokenId, amount, data)|
+---------------------------------------+
```

### MergingPool.sol

**Purpose:**

This contract allows users who participate in ranked battles, stake their NFTS to earn points and potentially win new fighter NFTs, this contract also gives the ecosystem the ability to control the levels of NFT inflation within the game.

**Key Features:**

- Users earn points for their fighters in ranked battles.
- Points are added to a merging pool.
- At the end of each period, winners are selected from the pool and awarded new fighters.
- Winners can claim their rewards as new fighter NFTs.
- Admins can manage the number of winners, select winners, and adjust other settings.

**Contract Structure:**

- **State Variables:** Store data like number of winners, current round ID, total points, owner address, ranked battle contract address, fighter farm contract instance, etc.
- **Mappings:** Track user claimed rounds, fighter points, winner addresses per round, winner selection status per round, and admin addresses.
- **Events:** Emit signals for points added and rewards claimed.
- **Functions:**
  - **External:** Public functions for transferring ownership, adjusting admin access, changing winners per period, picking winners (admins only), and claiming rewards.
  - **Public:** View functions for adding points (ranked battle contract only), retrieving fighter points, and getting unclaimed rewards.

**Additional Notes:**

- The contract interacts with the `RankedBattle` and `FighterFarm` contracts.
- Only the ranked battle contract can add points to fighters.
- Winners are randomly selected based on their points contribution to the pool.
- Users can claim multiple unclaimed rewards in a single transaction.

```css

+---------------------------------------+
| MergingPool - 110 sLOC |
+---------------------------------------+
| - winnersPerPeriod: uint256 |
| - roundId: uint256 |
| - totalPoints: uint256 |
| - _ownerAddress: address |
| - _rankedBattleAddress: address |
| - _fighterFarmInstance: FighterFarm |
| - numRoundsClaimed: mapping |
| - fighterPoints: mapping |
| - winnerAddresses: mapping |
| - isSelectionComplete: mapping |
| - isAdmin: mapping |
+---------------------------------------+
| + constructor(owner, rankedBattle, fighterFarm) |
| + transferOwnership(newOwner) |
| + adjustAdminAccess(admin, access) |
| + updateWinnersPerPeriod(newWinners) |
| + pickWinner(winners) |
| + claimRewards(modelURIs, modelTypes, customAttributes) |
| + getUnclaimedRewards(claimer) |
| + addPoints(tokenId, points) |
| + getFighterPoints(maxId) |
+---------------------------------------+
```

### Neuron.sol

**Purpose:**

This contract represents the ERC20 token for ArenaX Labs' AI Arena game, called Neuron (NRN). It manages NRN token creation, distribution, and transactions, this is cause it's implemented as the in-game economy's token

**Key Features:**

- ERC20 compliant token.
- Roles for minting, spending, and staking tokens.
- Initial mint for treasury and contributors.
- Maximum token supply.
- Ownership transfer and role management by owner.
- Airdrop setup for token distribution.
- Claiming tokens from the treasury.
- Minting and burning tokens (with appropriate roles).
- Approving spenders for stakers and other parties.

**Contract Structure:**

- **State Variables:** Store data like initial supply, treasury address, owner address, roles, etc.
- **Mappings:** Track admin status and token approvals.
- **Events:** Emit signals for claims and mints.
- **Functions:**
  - **External:** Public functions for transferring ownership, adding to roles, adjusting admin access, setting up airdrops, claiming tokens, minting tokens, and burning tokens.
  - **Public:** View functions for minting, burning, approving spenders, and burning from accounts.

**Additional Notes:**

- The contract interacts with other ArenaX Labs contracts like Ranked Battle and FighterFarm.
- Only specific roles can perform certain actions like minting or burning tokens.
- Users can claim NRN tokens from the treasury with an existing allowance.
- Admins can manage overall token distribution and roles.

```css

  +--------------------------------------+
  | Neuron - 92 sLOC |
  +--------------------------------------+
  | - MINTER_ROLE: bytes32 |
  | - SPENDER_ROLE: bytes32 |
  | - STAKER_ROLE: bytes32 |
  | - INITIAL_TREASURY_MINT: uint256 |
  | - INITIAL_CONTRIBUTOR_MINT: uint256 |
  | - MAX_SUPPLY: uint256 |
  | - treasuryAddress: address |
  | - _ownerAddress: address |
  | - isAdmin: mapping |
  +--------------------------------------+
  | + constructor(owner, treasury, contributor) |
  | + transferOwnership(newOwner) |
  | + addMinter(newMinter) |
  | + addStaker(newStaker) |
  | + addSpender(newSpender) |
  | + adjustAdminAccess(admin, access) |
  | + setupAirdrop(recipients, amounts) |
  | + claim(amount) |
  | + mint(to, amount) |
  | + burn(amount) |
  | + approveSpender(account, amount) |
  | + approveStaker(owner, spender, amount) |
  | + burnFrom(account, amount) |
  +--------------------------------------+
```

### RankedBattle.sol

**Purpose:**
This contract is in charge of providing functionality for staking NRN tokens on fighters, tracking their battle records, calculating and distributing rewards based on the outcomes of battle and staked amounts, this also allows claiming of accumulated rewards.

**Key Components:**

- **Staking and Rewards:** Users stake NRN tokens on fighters to earn points based on battle outcomes. Points determine NRN rewards.
- **Battle Record Tracking:** The contract tracks wins, ties, and losses for each fighter.
- **Round-Based System:** Rewards are distributed in rounds. Admins can set new rounds.
- **Stake at Risk:** Losing battles can put a portion of staked NRN at risk, temporarily reducing rewards until reclaimed through wins.
- **Merging Pool:** A portion of points is diverted to the merging pool for additional mechanics.

**Functionality:**

- **Staking NRN:** Users stake NRN on fighters using `stakeNRN`.
- **Unstaking NRN:** Users unstake NRN using `unstakeNRN`.
- **Claiming Rewards:** Users claim earned NRN rewards using `claimNRN`.
- **Updating Battle Records:** The game server updates battle records using `updateBattleRecord`, affecting points and NRN at risk.
- **Administrative Functions:** Admins can adjust settings like NRN distribution, basis points lost per loss, and start new rounds.

**Additional Notes:**

- **Voltage:** Initiating battles requires voltage, a separate resource tracked by the VoltageManager contract.
- **FixedPointMathLib:** The contract uses this library for fixed-point arithmetic.

**Key Points from Feedback:**

- **Clarity and Conciseness:** Present information in a clear and concise manner, avoiding unnecessary conversational elements.
- **Structure and Organization:** Organize information logically, using headings and bullet points for better readability.
- **Context and Examples:** Provide context for key concepts and examples to illustrate how the contract functions.

```css
+--------------------------------------+
| RankedBattle - 300 sLOC |
+--------------------------------------+
| - VOLTAGE_COST: uint8 |
| - totalBattles: uint256 |
| - globalStakedAmount: uint256 |
| - roundId: uint256 |
| - bpsLostPerLoss: uint256 |
| - _stakeAtRiskAddress: address |
| - _ownerAddress: address |
| - _gameServerAddress: address |
| - _neuronInstance: Neuron |
| - _fighterFarmInstance: FighterFarm |
| - _voltageManagerInstance: VoltageManager |
| - _mergingPoolInstance: MergingPool |
| - _stakeAtRiskInstance: StakeAtRisk |
| - isAdmin: mapping |
| - fighterBattleRecord: mapping |
| - amountClaimed: mapping |
| - numRoundsClaimed: mapping |
| - accumulatedPointsPerAddress: mapping |
| - accumulatedPointsPerFighter: mapping |
| - totalAccumulatedPoints: mapping |
| - rankedNrnDistribution: mapping |
| - hasUnstaked: mapping |
| - amountStaked: mapping |
| - stakingFactor: mapping |
| - _calculatedStakingFactor: mapping |
+--------------------------------------+
| + constructor(owner, gameServer, fighterFarm, voltageManager) |
| + transferOwnership(newOwner) |
| + adjustAdminAccess(admin, access) |
| + setGameServerAddress(newServer) |
| + setStakeAtRiskAddress(newStakeAtRisk) |
| + instantiateNeuronContract(nrnAddress) |
| + instantiateMergingPoolContract(mergingPoolAddress) |
| + setRankedNrnDistribution(newDistribution) |
| + setBpsLostPerLoss(newBpsLost) |
| + setNewRound() |
| + stakeNRN(amount, tokenId) |
| + unstakeNRN(amount, tokenId) |
| + claimNRN() |
| + updateBattleRecord(tokenId, mergingPortion, battleResult, eloFactor, initiatorBool) |
| + getBattleRecord(tokenId) |
| + getCurrentStakingData() |
| + getNrnDistribution(roundId) |
| + getUnclaimedNRN(claimer) |
+--------------------------------------+
```

### StakeAtRisk.sol

**Purpose:**

- Manages NRN tokens placed at risk during Ranked Battles.
- Stores and updates NRN amounts at risk for each fighter in each round.
- Enables reclaiming NRN after wins and sweeping lost NRN to the treasury at round end.

**Components:**

- **State Variables:**
  - `roundId`: Tracks the current Ranked Battle round.
  - `treasuryAddress`: Address of the NRN treasury.
  - `_rankedBattleAddress`: Address of the `RankedBattle` contract.
  - `_neuronInstance`: Instance of the `Neuron` contract (managing NRN tokens).
  - **Mappings:**
    - `totalStakeAtRisk`: Stores total NRN at risk for each round.
    - `stakeAtRisk`: Tracks NRN at risk for each fighter in each round.
    - `amountLost`: Records the total NRN lost by each address.

**Functionality:**

- **`setNewRound` (External):**
  - Called by `RankedBattle` to start a new round.
  - Sweeps all NRN at risk from the previous round to the treasury.
  - Updates `roundId`.
- **`reclaimNRN` (External):**
  - Called by `RankedBattle` when a fighter wins while having NRN at risk.
  - Transfers reclaimed NRN back to `RankedBattle`.
  - Updates `stakeAtRisk` and `totalStakeAtRisk`.
  - Emits `ReclaimedStake` event.
- **`updateAtRiskRecords` (External):**
  - Called by `RankedBattle` when a fighter loses with 0 points (enters deficit).
  - Moves part of their staked NRN to `stakeAtRisk`.
  - Updates `stakeAtRisk`, `totalStakeAtRisk`, and `amountLost`.
  - Emits `IncreasedStakeAtRisk` event.
- **`getStakeAtRisk` (External):**
  - Returns the NRN at risk for a specific fighter in the current round.
- **`_sweepLostStake` (Private):**
  - Transfers all NRN at risk from the current round to the treasury.

**Key Features:**

- Only the `RankedBattle` contract can interact with `StakeAtRisk`.
- NRN at risk is swept to the treasury at the end of each round, preventing accumulation.
- The contract tracks individual lost NRN amounts for potential future use.

**Addressing Feedback:**

- **Clarity and Conciseness:** Removed unnecessary conversational elements and focused on essential information.
- **Structure and Organization:** Used headings and bullet points for clarity.
- **Context and Examples:** Explained concepts like "deficit" and the relationship with `RankedBattle`.

```css
+-------------------------------------+
| StakeAtRisk - 63 sLOC |
+-------------------------------------+
| - roundId: uint256 |
| - treasuryAddress: address |
| - _rankedBattleAddress: address |
| - _neuronInstance: Neuron |
| - totalStakeAtRisk: mapping |
| - stakeAtRisk: mapping |
| - amountLost: mapping |
+-------------------------------------+
| + constructor(treasuryAddress, nrnAddress, rankedBattleAddress) |
| + setNewRound(roundId) |
| + reclaimNRN(nrnToReclaim, fighterId, fighterOwner) |
| + updateAtRiskRecords(nrnToPlaceAtRisk, fighterId, fighterOwner) |
| + getStakeAtRisk(fighterId) |
+-------------------------------------+
```

### VoltageManager.sol

**Purpose:**

- Manages the "voltage" resource used in the AI Arena game.
- Tracks voltage amounts for players and allows spending/replenishing.
- Controls who can spend voltage on behalf of players.

**Key Components:**

- **State Variables:**
  - `_ownerAddress`: Address with ownership privileges (initially deployer).
  - `_gameItemsContractInstance`: Instance of the `GameItems` contract.
  - **Mappings:**
    - `allowedVoltageSpenders`: Tracks addresses allowed to spend voltage for others.
    - `ownerVoltageReplenishTime`: Stores the next voltage refill time for each address.
    - `ownerVoltage`: Tracks current voltage amount for each address.
    - `isAdmin`: Indicates admin status for addresses.
- **Functions:**
  - **External:**
    - `transferOwnership`: Transfers ownership to a new address (owner only).
    - `adjustAdminAccess`: Grants/revokes admin privileges (owner only).
    - `adjustAllowedVoltageSpenders`: Controls who can spend voltage for others (admins only).
  - **Public:**
    - `useVoltageBattery`: Replenishes voltage by burning a battery item.
    - `spendVoltage`: Spends voltage for the caller or an allowed spender.
  - **Private:**
    - `_replenishVoltage`: Replenishes voltage and sets the next refill time.

**Key Features:**

- Only the owner, admins, or allowed spenders can modify voltage.
- Voltage automatically replenishes to 100 every 24 hours.
- Spending voltage requires having at least 100 voltage.
- The `GameItems` contract is used for managing battery items.

**Addressing Feedback:**

- **Clarity and Conciseness:** Used simpler language and focused on key concepts.
- **Structure and Organization:** Grouped related information and used bullet points.
- **Context and Examples:** Explained voltage usage and admin roles in context.

```css
+-------------------------------------+
|         VoltageManager - 47 sLOC     |
+-------------------------------------+
| - _ownerAddress: address            |
| - _gameItemsContractInstance: GameItems |
+-------------------------------------+
| - allowedVoltageSpenders: mapping   |
| - ownerVoltageReplenishTime: mapping|
| - ownerVoltage: mapping             |
| - isAdmin: mapping                  |
+-------------------------------------+
| + constructor(ownerAddress, gameItemsContractAddress) |
| + transferOwnership(newOwnerAddress)|
| + adjustAdminAccess(adminAddress, access) |
| + adjustAllowedVoltageSpenders(allowedVoltageSpender, allowed) |
| + useVoltageBattery()               |
| + spendVoltage(spender, voltageSpent)|
+-------------------------------------+
| - _replenishVoltage(owner)          |
+-------------------------------------+

```

## Systemic Risks

- Possible issues with the ecosystem being on an L2 since the sequencer being down would caase all timing logics to now be faulty.
- Protocol does not consider all the publicly available trnasfer functions while setting limits on the transferablity of NFTs/tokenIds.
- External dependencies from pragma oracle, inherited contracts, etc.
- Incorrect implementations of supported EIPs, not following the CEI pattern and not using local variables for emitting packed storage variables.

## Centralization Risks

- An admin could as well just maliciously delete attributes they are not meant to.
- An admin could just update the URI to now point to a malicious one and have users affected.
- Spender role can actually approve themselves for anybody's tokens and burn them.

## Recommendations

- Enhance documentation quality, currently the code's natspec and protocol's provided docs, do not always align, for example in the docs, it's stated that the merging pool is to use the chainlink VRF, but this is no where implemented in the code since this piece of information is now updated.

- Onboard more developers, cause multiple eyes checking out a protocol can be invaluable. More contributors can significantly reduce potential risks and oversights. Evidence of where this could come in handy can be gleaned from codebase typos. Drawing from the [broken windows theory](https://en.wikipedia.org/wiki/Broken_windows_theory), such lapses may signify underlying code imperfections.

- Re-enhance event monitoring, current implementation subtly suggests that event tracking isn't maximized. Instances where events are missing or seem arbitrarily embedded have been observed. for example, considering [RankedBattle.sol#L417-L504](https://github.com/code-423n4/2024-02-ai-arena/blob/befe84a9ddccba0cd0251082046e57ecabffb0cf/src/RankedBattle.sol#L417-L504) one notices that the event on the changed points of a user s not always emitted which leads to more dificulties in regards to off-chain tracking since a user's points would even reach 0 and the event won't be emitted
- Improve protocol's testability, one thing to note about tests is that _there's always room for further refinement and improvement._, a few out of the box ideas need to also be attached to test cases, for example a user can currently side step the `MAX_FIGHTERS_ALLOWED` in `FighterFarm.sol` which one of the reason is cause multiple test cases where not applied to the transfers and as such not all publicly available functions that can transfer were tested to be protected.
- To support the above, even fuzz and invariant tests could be further incorporated to help secure protocol.
- There seems to be a need to improve the naming conventions in some areas of protocol to ease user/developer experience while trying to use/understand protocol.
- Two step variable and address updates should be implemented including zero address checks. A timelock can also be considered for the setter functions to give users time to react to protocol changes. Adding these fixes can help protect from mistakes and unexepected behaviour.
- Add more developers or team members.

## Security Researcher Logistics

My attempt on reviewing the Codebase spanned around 40 hours distributed over 8 days:

- 2 hours dedicated to writing this analysis.
- 5 hours (first day) spent exploring the whole system to grasp the foundations using the docs before diving into it line by line
- 3 hours over the course of the 8 days were spent on the [discord chat for the contest](https://discord.com/channels/810916927919620096/1201981655850426399) to get more ideas based on questions other security researchers ask and explanation provided by sponsors.(Some of this time were also spent in the private thread under this chat, created by me to have a discussion with the sponsors regarding potential vulnerabilities.)
- The remaining time was spent on finding issues and writing the report for each of them.

## Conclusion

The codebase was a very great learning experience, though it was a pretty hard nut to crack, nonetheless during my review, I uncovered a few issues within the protocol and they need to be fixed. Recommended steps should be taken to protect the protocol from potential attacks. Timely audits and codebase cleanup for mitigations should also be conducted to keep the codebase fresh and up to date with evolving security times.

## Resources

- [Audit Contest Page](https://github.com/code-423n4/2024-02-ai-arena/tree/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc)

- [Official Documentation](https://docs.aiarena.io/gaming-competition)

- [Website](https://aiarena.io/#/)

### Time spent:
40 hours