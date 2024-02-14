# AI Arena Protocol Analysis

# Table of Contents
- [1. Overview of the AI Arena Protocol](#1-overview-of-the-ai-arena-protocol)
    - [1.1. The Core Modules and Main Actors of the AI Arena Protocol](#1-1-the-core-modules-and-main-actors-of-the-ai-arena-protocol)
    
- [2. Analysis of the Protocol Components and Smart Contracts](#2-analysis-of-the-protocol-components-and-smart-contracts)
    - [2.1. The Fighter Component](#2-1-fighter-component)
      - [2.1.1. FighterFarm.sol](#2-1-1-fighterfarm-sol)
      - [2.1.2. AiArenaHelper.sol](#2-1-2-aiarenahelper-sol)
      - [2.1.3. FighterOps.sol](#2-1-3-fighterops-sol)

    - [2.2. The Battle Component](#2-2-battle-component)
      - [2.2.1. RankedBattle.sol](#2-2-1-rankedbattle-sol)
      - [2.2.2. MergingPool.so](#2-2-2-mergingpool-sol)
      - [2.2.3. StakeAtRisk.sol](#2-2-3-stakeatrisk-sol)
      - [2.2.4. VoltageManager.sol](#2-2-4-voltagemanager-sol)
      - [2.2.5. GameItems.sol](#2-2-5-gameitems-sol)
    
    - [2.3. The Protocol Token: Neuron - NRN](#2-3-neuron-token)
    

- [3. Systemic and Technical Risks](#3-systemic-and-technical-risks)

    - [3.1. Upgradeable Contracts](#upgradeable-contracts)
    - [3.2. Use of Third-Party Libraries - Ecrecover](#use-of-third-party-libraries-ecrecover)
    - [3.3. Unbounded Loops can cause an Out Of Gas Exception](#unbounded-loop)
    - [3.4. Potentially Unsupported PUSH0 Opcode on Arbitrum](#push0-arbitrum)

- [4. Centralization Risks](#4-centralization-risks)

- [5. Analysis of Protocol Testing Strategies](#5-analysis-of-protocol-testing-strategies)


<a id="1-overview-of-the-ai-arena-protocol"></a>
# 1. Overview of the AI Arena Protocol

AI Arena is a PvP fighting game that allows participants to purchase and train AI-enabled NFTs and to use their fighters in battles in order to gain **Neuron** tokens (NRN) and potentially other NFTs.

Players first need to obtain NRN tokens and stake them in order to participate in ranked battles. If a player wins a battle, he gains points, which can be converted into NRN tokens. However, if the player loses a battle he risks to lose some of his staked NRN tokens.

The fighters (NFTs) are trained through imitation learning and the goal is knock opponents off of a platform in ranked battles in order to gain maximum points.

What is particularly interesting about this game is the role the participant plays in the AI-enabled training process of the NFT. The user teaches the fighter character (NFT) through demonstration. This means the user teaches the NFT to react in specific ways to different circumstances and situations. Then the behavior of the NFT is analyzed, fine-tuned, and potentially modified...

The first generation of NFTs are powered by a type of machine learning model called neural networks. Each NFT character has a set of distinct properties that define the visual and battle attributes of the character, such as hair, eyes, mouth... and weight, power, speed...

Of course, this short introduction barely scratches the surface of the possibilities offered by the game. For a detailed description of the NFT properties and attributes, the AI-enabled training process, the battle arena, the NRN rewards system and the ranking mechanism of the game, please check out the official documents on: https://docs.aiarena.io/gaming-competition


<a id="1-1-the-core-modules-and-main-actors-of-the-ai-arena-protocol"></a>
## 1.1. The Core Modules and Main Actors of the AI Arena Protocol

The protocol consists of various smart contracts that interact with each other. There are basically two core modules: **Fighter and Battle**. Neuron (NRN) is the protocol token and it is used by almost all protocol smart contracts.

Also, there is an off-chain component, the Game Server, that runs the core game logic and updates the battle record for a fighter, by calling the **updateBattleRecord()** function on the **RankedBattle** contract.

**The protocol modules and smart contracts:**

* Fighter
	* FighterFarm
	* AIArenaHelper
	* FighterOps (library)

* Battle
	* RankedBattle
	* MergingPool
	* StakeAtRisk
	* VoltageManager
	* GameItems

* Neuron, the protocol token is used by both modules.


**The main actors of the protocol:**

* Owner - each contract has an owner, which is initially set to the deployer of the contract 

* Admin - most contracts also have an admin that always to modify various protocol specific parameters. The contract owner always has the admin role.

* User - those are the game participants


**The image below provides a high-level overview of all the protocol components and the actors involved:**

![Architecture](https://raw.githubusercontent.com/rspadinger/C4-AIArena/master/code/images/Architecture.png)


<a id="2-analysis-of-the-protocol-components-and-smart-contracts"></a>
# 2. Analysis of the Protocol Components and Smart Contracts

<a id="2-1-fighter-component"></a>
## 2.1. The Fighter Component

The contracts in this module allow the players to obtain pre-determined fighters (NFTs) by providing the corresponding signature(s). The user also has the option to burn one or several mint passes in exchange for fighter NFTs. There are various other functions, such as re-rolling an existing NFT, updating the NFT model data... which are discussed in the following sections.

**The Fighter component consists of the following smart contracts:**

* FighterFarm
* AIArenaHelper
* FighterOps (library)

The image below provides a high-level overview of the Fighter module with all smart contracts involved, the core features provided by each individual contract and all involved actors. A detailed description of each smart contract is provided in the following sections of the report.

![Fighter](https://raw.githubusercontent.com/rspadinger/C4-AIArena/master/code/images/Fighter.png)


<a id="2-1-1-fighterfarm-sol"></a>
### 2.1.1. FighterFarm.sol

**Source:** https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol

This is the main contract of the Fighter component. It represents an NFT of type ERC721 and it allows players to claim fighter NFTs, redeem a mint pass for an NFT, update the staking status of a fighter, modify model attributes and generate a fighter with random attributes from an existing NFT. 

**NOTE:** The most important imports, state variables and functions are listed for each smart contract. However, more generic items, such as imported contract references, common view functions, private functions... will not be listed.

#### Imports:

Various required protocol contracts (Neuron, AiArenaHelper...) and the ERC721 and ERC721Enumerable contracts from OpenZeppelin. 


#### State Variables: 

There are various fighter related state variables, such as: 

MAX_FIGHTERS_ALLOWED, maxRerollsAllowed, rerollCost, generation, totalNumTrained, fighters[]...

**And the following mappings:**

* (uint256 => bool) fighterStaked
* (uint256 => uint8) numRerolls
* (address => bool) hasStakerRole 
* (uint8 => uint8) numElements 
* (address => (uint8 => uint8)) nftsClaimed 
* (uint256 => uint32) numTrained 
* (uint256 => string)  _tokenURIs


A detailed list of all state variables with their visibility identifiers and default values can be found in the smart contract code on Github (see link above).


#### Functions:

The features provided by this contract have already been listed in the introduction of this section. Here is a list of the public/external functions with their arguments. The function names are self-explanatory, so no additional description is provided here. Also, typical setter functions (like: setSomeContractAddress...) are not listed and the function visibility is omitted.

**State changing functions:**

* transferOwnership(address newOwnerAddress)
* incrementGeneration(uint8 fighterType)
* addStaker(address newStaker)
* updateFighterStaking(uint256 tokenId, bool stakingStatus)

* claimFighters(uint8[2] numToMint, bytes signature, string[] modelHashes, string[] modelTypes)

* redeemMintPass(uint256[] mintpassIdsToBurn, uint8[] fighterTypes, uint8[] iconsTypes, string[] mintPassDnas, string[] modelHashes, string[] modelTypes)

* mintFromMergingPool(address to, string  modelHash, string  modelType, uint256[2]  customAttributes)

* updateModel(uint256 tokenId, string  modelHash, string  modelType)

* transferFrom(address from, address to, uint256 tokenId)

* reRoll(uint8 tokenId, uint8 fighterType)


**View functions:**

* doesTokenExist(uint256 tokenId) returns (bool)
* tokenURI(uint256 tokenId) returns (string)
* getAllFighterInfo(uint256 tokenId) returns (address, uint256[6], uint256, uint256, string, string, uint16)


<a id="2-1-2-aiarenahelper-sol"></a>
### 2.1.2. AiArenaHelper.sol

**Source:** https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol

As the name indicates, this is a helper contract that provides various state variables and functions for the FighterFarm contract. It allows to create the physical attributes of a fighter NFT and to return the attribute probabilities for a specific fighter generation.


#### State Variables: 

There are various state variables related to fighter attributes, such as: 

* attributes = ["head", "eyes", "mouth", "body", "hands", "feet"]
* defaultAttributeDivisor = [2, 3, 5, 7, 11, 13]


**And the following mappings:**

* (uint256 => mapping(string => uint8[]) attributeProbabilities
* (string => uint8) attributeToDnaDivisor;


#### Functions:

**State changing functions:**

* addAttributeDivisor(uint8[] attributeDivisors)
* addAttributeProbabilities(uint256 generation, uint8[][] probabilities)
* deleteAttributeProbabilities(uint8 generation)
* createPhysicalAttributes(uint256 dna, uint8 generation, uint8 iconsType, bool dendroidBool)


**View functions:**

* getAttributeProbabilities(uint256 generation, string attribute) returns (uint8[])



<a id="2-1-3-fighterops-sol"></a>
### 2.1.3. FighterOps.sol

**Source:** https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol

This is a helper library that provides various structs and functions for the FighterFarm contract. It allows to return the physical attributes and the NFT attributes (such as model, weight, generation...) for a specific fighter.


#### Structs: 

* FighterPhysicalAttributes(head, eyes, mouth, body, hands, feet)

* Fighter(weight, element, physicalAttributes, id, modelHash, modelType, generation, iconsType, dendroidBool)


#### Functions:

**View functions:**

getFighterAttributes(Fighter storage self) returns (uint256[6])

viewFighterInfo(Fighter storage self, address owner) returns (address, uint256[6], uint256, uint256, string, string, uint16)


<a id="2-2-battle-component"></a>
## 2.2. The Battle Component

The contracts in this module allow the players to stake their NRN tokens and to compete in ranked battled. If a player wins a battle, he can claim NRN reward tokens. If a player loses one or several battles, he risks to lose some of his staked NRN tokens.

There are various other functions, such as updateBattleRecord(), setNewRound(), updateAtRiskRecords(), spendVoltage(), createGameItem()... which are discussed in the following sections.


**The Battle component consists of the following smart contracts:**

* RankedBattle
* MergingPool
* StakeAtRisk
* VoltageManager
* GameItems

The image below provides a high-level overview of the Fighter module with all smart contracts involved, the core features provided by each individual contract and all involved actors. A detailed description of each smart contract is provided in the following sections of the report.

![Battle](https://raw.githubusercontent.com/rspadinger/C4-AIArena/master/code/images/Battle.png)


<a id="2-2-1-rankedbattle-sol"></a>
### 2.2.1. RankedBattle.sol

**Source:** https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol

This is the main contract of the Battle component that allows player to stake and unstake NRN tokens in order to participate in ranked battles and to claim reward tokens if the player wins one or more ranked battles.

As already mentioned, the core game logic is run by off-chain servers. The game server calls the updateBattleRecord() function on the this contract in order to update the battle record for a specific fighter.


#### State Variables: 

There are various battle related state variables, such as: 

VOLTAGE_COST, totalBattles, globalStakedAmount, roundId, bpsLostPerLoss...

**And the following mappings:**

* (uint256 => BattleRecord) fighterBattleRecord   
* (address => uint256) amountClaimed
* (address => uint32) numRoundsClaimed
* (address => mapping(uint256 => uint256)) accumulatedPointsPerAddress
* (uint256 => mapping(uint256 => uint256)) accumulatedPointsPerFighter
* (uint256 => uint256) totalAccumulatedPoints
* (uint256 => uint256) rankedNrnDistribution
* (uint256 => mapping(uint256 => bool)) hasUnstaked
* (uint256 => uint256) amountStaked
* (uint256 => uint256) stakingFactor
* (uint256 => mapping(uint256 => bool)) _calculatedStakingFactor


#### Functions:

The features provided by this contract have already been listed in the introduction of this section. Here is a list of the public/external functions with their arguments. The function names are self-explanatory, so no additional description is provided here. Also, typical setter functions (like: setSomeContractAddress...) are not listed and the function visibility is omitted.

**State changing functions:**

* setNewRound()
* updateBattleRecord(uint256 tokenId, uint256 mergingPortion, uint8 battleResult, uint256 eloFactor, bool initiatorBool)

* stakeNRN(uint256 amount, uint256 tokenId)
* unstakeNRN(uint256 amount, uint256 tokenId)
* claimNRN()


**View functions:**

* getBattleRecord(uint256 tokenId) returns (uint32, uint32, uint32)
* getCurrentStakingData() returns (uint256, uint256, uint256)
* getNrnDistribution(uint256 roundId_) returns (uint256)
* getUnclaimedNRN(address claimer) returns (uint256)


<a id="2-2-2-mergingpool-sol"></a>
### 2.2.2. MergingPool.sol

**Source:** https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol

The MergingPool contract allows the admin to pick the winner for the current round and the player to claim the rewards for one or more rounds. The RankedBattle contract is the only actor that can add points to a specific fighter.


#### State Variables: 

There are various state variables that allow to manage fighter points for a specific round, such as: 

winnersPerPeriod, roundId, totalPoints...


**And the following mappings:**

* (address => uint32)  numRoundsClaimed
* (uint256 => uint256)  fighterPoints
* (uint256 => address[])  winnerAddresses
* (uint256 => bool)  isSelectionComplete


#### Functions:

**State changing functions:**

* updateWinnersPerPeriod(uint256 newWinnersPerPeriodAmount)
* pickWinner(uint256[] winners)
* addPoints(uint256 tokenId, uint256 points)
* claimRewards(string[] modelURIs, string[] modelTypes, uint256[2][] customAttributes)


**View functions:**

* getUnclaimedRewards(address claimer) returns (uint256)
* getFighterPoints(uint256 maxId) returns (uint256[])


<a id="2-2-3-stakeatrisk-sol"></a>
### 2.2.3. StakeAtRisk.sol

**Source:** https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol

As the name indicates, this contract allows to manages the player NRN token stake that is put at risk. The RankedBattle contract can also reclaimNRN tokens from the stake at risk.  


#### State Variables: 

**There are the following mappings:**

* (uint256 => uint256) totalStakeAtRisk  
* (uint256 => (uint256 => uint256)) stakeAtRisk
* (address => uint256) amountLost


#### Functions:

**State changing functions:**

* setNewRound(uint256 roundId_)
* reclaimNRN(uint256 nrnToReclaim, uint256 fighterId, address fighterOwner)
* updateAtRiskRecords(uint256 nrnToPlaceAtRisk, uint256 fighterId, address fighterOwner)


**View functions:**

* getStakeAtRisk(uint256 fighterId) external view returns (uint256


<a id="2-2-4-voltagemanager-sol"></a>
### 2.2.4. VoltageManager.sol

**Source:** https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol

This contract manages the voltage of the NFT fighter character.
 
#### State Variables: 

**The contract contains the following mappings:**

* mapping(address => bool) allowedVoltageSpenders
* mapping(address => uint32) ownerVoltageReplenishTime
* mapping(address => uint8) ownerVoltage


#### Functions:

**State changing functions:**

adjustAllowedVoltageSpenders(address allowedVoltageSpender, bool allowed)
useVoltageBattery()
spendVoltage(address spender, uint8 voltageSpent)






<a id="2-2-5-gameitems-sol"></a>
### 2.2.5. GameItems.sol

**Source:** https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol

This is an ERC1155 contract, which manages a collection of game items.
 

#### Imports:

The ERC1155 contract from OpenZeppelin. 


#### State Variables: 

The contract uses a struct for the game item attributes and various game item related storage variables, such as: itemCount, treasuryAddress, allGameItemAttributes[]...

struct GameItemAttributes {
        string name;
        bool finiteSupply;
        bool transferable;
        uint256 itemsRemaining;
        uint256 itemPrice;
        uint256 dailyAllowance;
 }


**And the following mappings:**

* (address => mapping(uint256 => uint256)) allowanceRemaining
* (address => mapping(uint256 => uint256)) dailyAllowanceReplenishTime
* (address => bool) allowedBurningAddresses
* (address => bool) isAdmin
* (uint256 => string) _tokenURIs;


#### Functions:

**State changing functions:**

* adjustTransferability(uint256 tokenId, bool transferable)
* setAllowedBurningAddresses(address newBurningAddress)
* setTokenURI(uint256 tokenId, string memory _tokenURI)
* createGameItem(string name_, string tokenURI, bool finiteSupply, bool transferable, uint256 itemsRemaining, uint256 itemPrice, uint16 dailyAllowance) 

* mint(uint256 tokenId, uint256 quantity)
* burn(address account, uint256 tokenId, uint256 amount)
* safeTransferFrom(address from, address to, uint256 tokenId, uint256 amount, bytes data)


**View functions:**

* contractURI() returns (string)
* getAllowanceRemaining(address owner, uint256 tokenId) returns (uint256)
* remainingSupply(uint256 tokenId) returns (uint256)
* uniqueTokensOutstanding() returns (uint256)


<a id="2-3-neuron-token"></a>
## 2.3. The Protocol Token: Neuron - NRN

This is the protocol token. Users need to obtain and stake NRNs in order to participate in ranked battles. Battle winners are rewarded with NRN tokens. This is a standard ERC20 token with some additional state variables and functions in order to manage the Spender, Staker and Minter roles, to mint a certain amount of tokens for the treasury and the contributor and to setup the allowance for an airdrop.

Most contract in the Fighter and Battle components make use of the Neuron contract.


#### Imports:

The ERC20 and AccessControl contracts from OpenZeppelin. 


#### State Variables: 

The contract uses the following state variables, which are all defined as public constant:

* bytes32 MINTER_ROLE = keccak256("MINTER_ROLE")
* bytes32 SPENDER_ROLE = keccak256("SPENDER_ROLE")
* bytes32 STAKER_ROLE = keccak256("STAKER_ROLE")

* uint256 INITIAL_TREASURY_MINT = 10 ** 18 * 10 ** 8 * 2
* uint256 INITIAL_CONTRIBUTOR_MINT = 10 ** 18 * 10 ** 8 * 5
* uint256 MAX_SUPPLY = 10 ** 18 * 10 ** 9


#### Functions:

**State changing functions:**

* addMinter(address newMinterAddress)
* addStaker(address newStakerAddress)
* addSpender(address newSpenderAddress)
* adjustAdminAccess(address adminAddress, bool access)
* setupAirdrop(address[] recipients, uint256[] amounts)
* mint(address to, uint256 amount)
* approveSpender(address account, uint256 amount)
* approveStaker(address owner, address spender, uint256 amount)

* claim(uint256 amount)
* burn(uint256 amount)


<a id="3-systemic-and-technical-risks"></a>
# 3. Systemic and Technical Risks


<a id="upgradeable-contracts"></a>
## 3.1. Upgradeable Contracts

The contracts of the protocol are currently not upgradeable. This means, if at a later stage bugs need to be fixed or additional features need to be rolled out, things will be more complicated than they would be if the protocol would have been designed with upgradeability in mind:

* More effort is required to deploy a new version
* The rollout needs to be planned very carefully
* Current users need to be informed about the protocol change
* Websites, script... that use the old contracts will be outdated and all contract references need to be modified
* There could be some downtime where the protocol is not accessible
* Gaming applications are more prone to upgrades than other type of protocols (like a DEX or a borrowing/lending protocol...). Therefore, making the protocol upgradable could add some significant advantages. 


On the other hand, rolling out "upgradeable" smart contracts goes somewhat against the idea of building decentralized web3 solutions that are "truly" different from traditional web2 applications. If a team can easily upgrade a protocol and change whatever they want and whenever they see fit, this vision is no longer fulfilled.

The team needs to weigh up all advantages and disadvantages of both approaches.

**OpenZeppelin provides various libraries and contracts that make it relatively easy to make a protocol upgradeable. The two most common patterns are:**

* Transparent proxy: https://docs.openzeppelin.com/contracts/5.x/api/proxy#transparent_proxy 

* And, the Universal Upgradeable Proxy Standard (UUPS): https://docs.openzeppelin.com/contracts/5.x/api/proxy#UUPSUpgradeable



<a id="use-of-third-party-libraries-ecrecover"></a>
## 3.2. Use of Third-Party Libraries - Ecrecover

The claimFighters() function in the FighterFarm contract calls the verify() function in the Verification contract, which uses the Solidity ecrecover method.

This function is known to be vulnerable to signature malleability and OpenZeppelin's ECDSA helper library should be used instead:

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol 


<a id="unbounded-loop"></a>
## 3.3. Unbounded Loops can cause an Out Of Gas Exception

Unbounded loops are loops without a defined endpoint, meaning they can theoretically run indefinitely. In Solidity smart contracts, these loops can be particularly problematic, as they consume an unpredictable amount of gas. At a certain stage, the block.gasLimit will be reached, the transaction will fail and the entire protocol is at risk of becoming unusable. 

The claimFighters() function in the FighterFarm contract mints an undetermined number of fighter NFTs in an unbounded loop, which could potentially cause an out of gas exception.

The risk of such an exception in the claimFighters() function, however is very low as the user would need to mint a large amount of NFTs for which he also needs a valid signature.

However, to be safe, a verification should be added to the claimFighters() function that limits the number of NFTs that can be minted in one batch.


<a id=" "></a>
## 3.4. Potentially Unsupported PUSH0 Opcode on Arbitrum

The protocol contracts use an unspecified pragma version, which is not recommended:

pragma solidity >=0.8.0 <0.9.0;

As the protocol is destined to be deployed on the Arbitrum L2 network, which does not yet support the PUSH0 opcode, Solidity versions higher than 0.8.19 should not be used for the protocol.

If a Solidity version higher than 0.8.19 is used, it may not be possible to deploy the contracts to Arbitrum and even if the deployment succeeds, there is a risk that the contracts will not function properly. 



<a id="4-centralization-risks"></a>
# 4. Centralization Risks

While contract ownership and standard access role management can greatly simplify the execution of specific administrative actions it also increases centralization and can potentially add some substantial risks to the protocol. In a worst case scenario, this could even lead to the loss of all protocol funds and/or render the protocol unusable.

Key security features, like changing the contract owner should never be executed by a single external account. The new owner could be assigned to a non-existent account, the account private key could get lost or fall into the hands of a malicious actor... which would put the entire protocol at risk. 

**Instead, a 2-step approach should be used to change the owner of the contract:**

In the first step, the current owner should call a method to transfer the ownership. And, in the second step, the new owner needs to call a method to accept the ownership. 

The easiest way to implement this is by using Ownable2Step.sol contract from OpenZeppelin, which provides the transferOwnership(address newOwner) function to initiate the transfer and the function acceptOwnership() that needs to be called by the pending owner in order to accept the ownership of the contract and to finalize the ownership transfer.


<a id="5-analysis-of-protocol-testing-strategies"></a>
# 5. Analysis of Protocol Testing Strategies

The protocol has been tested extensively using a variety of unit test for all protocol smart contracts. However, there is a lack of integration tests.

Admittedly, there are not that many interesting scenarios for integration tests. However, the following scenario could provide an interesting test case:

•	Setup several players
•	Let them stake different amounts of NRN tokens
•	Perform several mock call of the updateBattleRecord() function for all participating fighters to simulate battle results without having to rely on the off-chain GameServer
•	Assert the fighterPoints for each player in the mergingPool
•	Assert the correct winner addresses in the mergingPool
•	Assert the stake at risk for each player
•	Assert that reward NFTs have been minted for winners with the correct attributes (modelHash, modelType, customAttributes...) 


**The current test coverage is at 94.6% in terms of lines of code tested. The image below provides a detailed overview of the test coverage for each smart contract file:**

![TestCoverage1](https://raw.githubusercontent.com/rspadinger/C4-AIArena/master/code/images/coverage.png)



### Time spent:
35 hours