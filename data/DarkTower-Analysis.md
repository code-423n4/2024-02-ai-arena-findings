Analysis Report

**Index table of content**
| S/NO  | Particulars  |
|---|---|
| 1  | Preface  |
| 2  | Conceptual overview  |
| 3  | Architecture (inclusive of a Diagram)  |
| 4  | Approach taken in evaluating the codebase (inclusive of Diagrams)  |
| 5  | Centralization risks  |
| 6  | Mechanism & Architecture review  |
| 7  | Architecture Recommendations   |


## 1. Preface
This analysis report has been approached with the following key points and goals in mind:

- Assumes not so tech-savvy users will stumble upon it and does not include redundant documentation the protocol team is already aware of. Instead, it breaks down the contracts in easy bits to grasp before jumping right in.
- This report's goal in part is to provide the protocol team valuable insights on some edge cases we have been able to find within the timeframe of audit and in part to help users digest the protocol bits.
- In the scenarios that we may have included some repetitive documentation, it is to provide the judge context on what the protocol team's assumptions are and therefore helps the judge grasp scenarios we cover better.

## 2. Conceptual overview
AI Arena is a GameFi protocol designed to entertain duel competitions onchain. Each round, two or more fighters (NFTs) do battle and the winner (if they perform well and put up some NRN stake themselves) wins some more NRN tokens; the staple AI Arena ERC20 token. Think of it as Fight Club, but with NFTs onchain. In a booming industry of GameFi protocols and moderately expensive transaction fees on Ethereum still, the protocol chose a great alternative blockchain to run on and help players manage resource costs - that blockchain being Arbitrum One. One significant challenge GameFi projects prior have faced is the limited character features of NFTs. AI Arena addresses this a number of ways but one of the most exciting feature is the ability to refill the voltage of a fighter through the `VoltageManager` contract. This ensures that when an NFT fighter becomes depleted and as a result become unable to fight, you just don't discard the fighter; instead you can fill it's voltage back up as well as do more training on it by switching/opting for better rarer attributes. Not to mention, all of this fight club individual performances is incentived by earning NRN tokens, the players will always look to build up on wins for a particular fighter NFT. Rather than mint, utilize and discard, players will look to mint, utilize, rebuild, utilize and continue that cycle.


## 3.  Architecture (Diagram)

![Excalidraw FlowChart](https://rexjoseph.github.io/images/ai-arena-entry.png)

Please visit the link to the full diagram view: https://rexjoseph.github.io/images/ai-arena-entry.png

This whole protocol has a single entry point from it's launch: the `redeemMintPass()` function.

This function allows users who have already gotten a hold of mint-passes to burn those mint passes in exchange for fighter NFT tokens when the protocol launches. This is a great feature. Unlike what you'd normally expect from a GameFi protocol that allows aggressive minting of NFTs at launch, AI Arena is looking to keep volume of NFTs relatively low at the beginning.


## 4. Approach taken in evaluating the codebase 
We employed a simple call path trace review process in looking at each in-scope contracts of the codebase - exploring core functions. Instead of simply stating our research day schedules and what we did each day, we find that giving you an understanding of those contracts we covered each day is a better review process for all parties involved, so we covered the contracts below:

1. FighterFarm.sol
2. RankedBattle.sol
3. MergingPool.sol
4. AiArenaHelper.sol
5. StakeAtRisk.sol
6. VoltageManager.sol
7. Neuron.sol
8. GameItems.sol

### **1. FighterFarm.sol**
Core contract, tt houses the entry point for users into the system. With 245 lines of code, it does a ton of logic such as managing creation, ownership, mints and redemptions of fighter NFTs. It extends the ERC721 implementation of Openzeppelin so you'd typically expect to find some unsafe external call behaviours from this contract engineered by players. Coupled with the `MergingPool` contract, it is vulnerable to unsafe behaviours as we would later see.

But you can already think of this contract as the starting point for getting into the arena, the contract you and your opponents will first interact with to get your fighters.

Some crucial variables are:
- `uint8 public constant MAX_FIGHTERS_ALLOWED` which defaults to 10 fighters maximum per player address.

- `uint8[2] public maxRerollsAllowed` which enforces a bound on the max reRolls for each NFT fighter.

- `uint256 public rerollCost` which keeps track of the amount of NRN tokens it will cost a player to reRoll a fighter NFT.

Below we take a deep dive of some crucial functions in this contract.

*`incrementGeneration()`*
Remember Pokemon generations? This function is similar. Different generations can exist at a time for a fighter type (fighter types are typically either champion or dendroid) and this function allows the contract owner to increment such generation for a fighter. It is crucial that players don't increment generations arbitrarily, so it's utilizing a centralization privilege here. Upon incrementation, the maximum number of reRolls allowed for the fighter type is bumped up by 1 more free reRoll.

*`claimFighters()`*
This function as it entails allows players to claim fighter NFTs. They do this by submitting a signature signed by the `_delegatedAddress`. This is a sensitive function in the sense that if implemented incorrectly, it will allow players claim more than once using the same signature. As a player, you provide a signature signed by the `_delegatedAddress` as well as the model hashes and types for the fighter NFTs you want to be minted which are then minted to your address after verifying that the provided signature is valid. This function directly calls `_createNewFighter()` which is the function where minting ensues and ensures each player don't breach the hard cap of 10 NFTs per address.

*`redeemMintPass()`*
The entry point for this protocol at the time of launch. This function allows users who have already acquired mint passes pre-launch to redeem such mint passes for fighter NFTs. In order to make sure you don't mint two NFTs for 1 mint pass, it burns each mint pass before minting to your address. This is another sensitive function in the sense that if implemented wrongly, players can circumvent the 1-1 mints of NFTs to mint passes.

*`reRoll()`*
This function allows the fighter NFT owner to reRoll their fighter in the hopes of probably getting a better trait. Traits are random and each fighter NFT has a pre-set maximum number of rolls they can utilize to switch to another trait. Though this function allows implement this functionality, it is not free. First, you can't have reached the max rolls, second you must have sufficient NRN tokens to pay for the attributes you get during a roll. The NRNs are then transferred to the treasury address while you get to keep your new fighter NFT trait.

*`_createNewFighter()`*
Private function where all the fighter NFT token mints happen. All external functions exposed to users for minting new fighter tokens interact with this core function. This function is where the requirement for not breaching the address cap of maximum fighters allowed occurs. It does some pre-determinations based on your input from the calling functions to determine which attributes, generation type and traits your fighter NFT to mint will have at which point it proceeds to mint the NFTs to your address.


### **2. RankedBattle.sol**
This smart contract holds the most code lines of the AI Arena protocol - most of the interesting functions that is. With 277 lines of code, it serves as the contract for managing fighter reward distributions, fighter NFT track records and staking of NRNs on fighters during battle duels. You can think of it as the second contract you interact with after accruing and building fighters with full voltages.

Some crucial variables are:
- `struct BattleRecord` which is a datastructure for keeping track of wins, losses and ties for a fighter.

- `uint8 public constant VOLTAGE_COST` set as 10 voltages per duel you initiate/join. After 10 matches, your voltage will be completely depleted.

- `uint256 public totalBattles` keeps track of the number of total battles that have been initiated.

- `mapping(uint256 => BattleRecord) public fighterBattleRecord` returns a `BattleRecord` datastructure for a given fighter ID.

Let's have a deep dive into some of the crucial functions this contract allows players to interact with.

*`setGameServerAddress()`*
This function allows the contract owner set the game server address that is responsible for updating game records and delivering results onchain. This address being set is similar to an oracle that feeds offchain data to some dependent contract onchain. You can then think of this address to act similar to the likes of a Chainlink Oracle.

*`setStakeAtRiskAddress()`*
The `StakeAtRisk` contract which manages staking NRNs for fighters during a duel needs be set as you would expect in order to facilitate that very task when a round ensues.

*`setNewRound()`*
Rounds are initiated one at a time and only permissioned admin addresses are allowed to do this. When a round ends, lost NRN tokens are swept to the protocol's treasury address and winners are rewarded with corresponding NRN tokens according to their points. When this round is set the current round ID is incremented by 1 to move us past the last round.

*`stakeNRN()`*
One crucial thing to note is that if you have unstaked for the current round before, it doesn't let you come back in to pledge a stake. Rather weird, but we can understand the logic for this as it means if you get out of the game with your stake, you won't be allowed back in for that fighter. You must hold the equivalent amount of NRN tokens you intend to stake or more in your wallet to ensure this transaction is a success.

*`unstakeNRN()`*
Opposite of calling `stakeNRN()`, this function unstakes NRN tokens from a fighter NFT they have currently staked for in the current game round.

*`claimNRN()`*
This function handles claiming of NRN tokens won for a specified round(s). Winners are allowed to claim only once per round they have been selected a winner for and no more.

*`updateBattleRecord()`*
This function has a strict enforcement that calls must come from the game server address/oracle. When called by such address, the function updates the battle record for a specific fighter NFT such as the current battle result (which will be win, lose or a tie), the total battles the fighter has fought etc. If this function had been exposed to anyone, some players will 90% of the time call it to update perhaps a leading fighter with bad data. The AI Arena protocol gates this function to ensure only valid records supplied by the server address can be used for each fighter NFT battle record.

### **3. MergingPool.sol**
105 lines of code. But plenty of interesting logic within those lines.
This contract handles a similar logic to airdropping in the sense that it allows a player to potentially earn a new fighter. Let's dive right in and explore some of it's crucial variables first:

- `uint256 public winnersPerPeriod` storage variable to set the bound for number of winners per period. This variable can be updated at any time by an admin. Initially, it starts off as two winners per period.

- `uint256 public roundId` storage variable to keep track of the current round ID; set to 0 which is a little redundant to set a default 0 value to another 0 value. The protocol team can just leave it uninitialized instead.

- `mapping(uint256 => address[]) public winnerAddresses` keeps track of the winner addresses in a specified round.

- `mapping(uint256 => bool) public isSelectionComplete` crucial and keeps track of whether or not a round's winners selection is completed.

Some crucial functions include:
*`updateWinnersPerPeriod()`*
Allows an admin to change the number of winners to be picked per round.

*`pickWinner()`*
When a round's winners selection is due, an admin calls this function to pick winners. Assume Bob and Alice are winners for rounds 0 & 1. When this function is executed, it means both Bob and Alice are entitled to 2 fighter NFTs for both rounds. These NFTs can be minted in the next function we will explore. When the winners are selected, the current round's selection completion is set to true to reflect this change and the `roundId` is increased.

*`claimRewards()`*
This is the function Bob and Alice will call to claim their won NFT fighters. This function mints fighter NFTs from the `MergingPool` contract provided that the caller is indeed a winner for the rounds. Each winner is entitled to the amount they are due and no more than which means each winner will typically get the same amount of fighter NFTs - if Bob gets 2, Alice gets 2.


### **4. AiArenaHelper.sol**
With just 90 lines of code, this contract allows management of fighter attributes such as the `head`, `eyes`, `mouth`, `body`, `hands` and `feet` etc.

Core functionalities are:

*`createPhysicalAttributes()`*
Allows players to create physical attributes for a fighter based on the supplied DNA. This function when supplied with the necessary arguments, proceeds to create the physical attributes utilizing the `FighterOps` library contract which we will not have a deep dive for in this analysis as it's pretty minimal.

*`addAttributeProbabilities()`*
Allows the functionality to add attribute probabilities for a given generation ID. This function is centralized and only accessible to the contract owner. 

Another core function of this contract, is the *`deleteAttributeProbabilities()`* function which does the opposite of the `addAttributeProbabilities` function logic - allowing deletions of attribute probabilities for a given generation ID.


### **5. StakeAtRisk.sol**
This contract plays a vital role in the AI Arena protocol as it is the contract to manage NRN tokens that are at risk of being swept in the case of a loss during a battle round. Some crucial functions we will cover are:

*`setNewRound()`*
In order to set a new staking round, the `RankedBattle` contract has to be the address calling this function to run the transaction. When this function is called, the previous staked and lost NRN tokens escrowed in this contract needs to be transferred out, so it calls the `_sweepLostStake()` function to transfer the lost NRN balances to the treasury address afterwhich it initializes a new round.

*`reclaimNRN()`*
Allows a fighter the opportunity to regain their staked NRN tokens against a fighter NFT. The call to this transaction must be from the `RankedBattle` contract which is done after the battle records update for a specific fighter has been carried out. It transfers the NRN tokens to reclaim to the `RankedBattle` contract and ensures if the transaction is successul, the internal balances of this contract's stakes at risk for a fighter, round, and amount lost are adjusted to reflect the new state resulting from this action.

*`_sweepLostStake()`*
The function that handles the NRN tokens at risk transfer to the treasury address. These are typically tokens lost in duel round being swept from this contract's balance.


### **6. VoltageManager.sol**
Implementation contract for managing power/voltage refills for players in the AI Arena protocol. It exposes some functions to use voltage as well as replenish it.

*`useVoltageBattery()`*
When this function is called, it makes sure the current voltage for the player/msg.sender is lesser than 100 (has been used atleast). Then it burns a game item from the player and replenishes the players voltage back up to a hundred (full). You can think of it like an in-game playstore where you typically would expect to spend dollars to replenish your battery or wait 24hrs for it to automatically come back up.

*`spendVoltage()`*
This functions allows a player to spend their just replenished voltage. You specify how much voltage you want to use from the player's address and you must be authorized to spend voltages on behalf of the player or be the player yourself for this function to not revert. Then it proceeds to log and decrement your voltage by the amount being spent. If the last time you refilled your voltage is 24hrs ago, if first proceeds to refill it back up to 100%.

*`_replenishVoltage()`*
This function does the logic for replenishing voltages. Voltages can only be replenished every 24 hours and the timelock for that is done within this function when called from the `spendVoltage()` function.


### **7. Neuron.sol**
The second most crucial contract in our opinion for this whole protocol to be exciting. This contract houses the NRN token implementation that fuels the AI Arena protocol. It has most of the ERC20 specification features you would expect from a token such as the `burn`, `mint` functions to engineer token supply alterations.

Two interesting and unsual functions we would like to deep dive into for this contract are the `setupAirdrop()` & `claim()` functions.

*`setupAirdrop()`*
Allows an admin to setup NRN token airdrop for an array of addresses and set's the amount of tokens each one of these addresses can claim. Ideally, for an airdrop you would expect a function that allows an admin to run the distribution but the protocol has opted to use a `claim()` mechanism to implement this. If the claim is implemented wrongly, it will poses vulnerabilities that allows some whitelisted address to double or thriple claim. Even worse, it could possibly allow non-whitelisted address to claim NRN tokens from an airdrop.

*`claim()`*
The function that allows a whitelisted address to claim the tokens they're approved for from the airdrop. Implemented very simply and clean.


### **8. GameItems.sol**
This contract facilitates the collection creation of game items and management in the AI Arena Protocol. Let's dive deep into the nitty gritty for some interesting functions.

*`mint()`*
If a game item has been created, that is, it exists as a collection, the caller of this function will be able to mint such game item as an ERC1155 token by paying the correct amount of NRN tokens it costs to mint the game item.

*`createGameItem()`*
Allows the functionality of new game items creation and only accessible to an admin. This function takes a couple of arguments such as the name for the game item, the token URI, whether or not it should have a finite (capped) supply, if it should be transferable between addresses (players), the number of items remaining (in the case it should have a finite supply), item price (in NRN tokens), and the daily allowance for minting such item. Then it uses all of these arguments to create the game item and opens the floor for purchases of it.

*`burn()`*
Allows a predetermined address with burner privilege to burn game items from an address.


## 5. Centralization risks
The functions listed below have accessibility prohibited to the `_ownerAddress`, `isAdmin`, `allowedBurningAddresses`, `hasStakerRole` addresses that can cause potential issues resulting from too much power concentration. Notably, these issues are more apparent in the `GameItems` contract for example where the creation of a game item is solely controlled by an admin address or the `burn` function which can engineer a burn across any address. Perhaps allowing addresses to burn their own items at will as well will be great. Perhaps allow a fighter the ability to adjust staking for their fighter in the future. Perhaps users should be able to approve others to spend NRN tokens on their behalf instead of only addresses that have the `SPENDER_ROLE`.

- [`burn()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L242)
- [`createGameItem()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L208)
- [`updateFighterStaking()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L269)
- [`approveSpender()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L173)


## 6. Mechanism & Architecture review 
Upon successful review of this codebase, we concluded that the AI Arena Protocol re-imagines GameFi in another perspective not yet widely adopted, "Player vs Player" while implementing a couple of unique features, all of which ensures fighter NFTs will not just be minted and discarded or left in addresses to rot. Instead, encouraging a competitive arena where fighter NFTs are continually advanced by their owners.

**Comparison between AI Arena and Axie Infinity**
| Feature  | AI Arena  | Axie Infinity  |
|---|---|---|
| Goal  | A more decentralized Player vs Player protocol empowering one-to-one duels within the community | An all-in-one GameFi protocol pioneering Play to Earn duels within the community |
| ERC20 Tokens  | Earned only if a player stakes it against their opponent in battle and wins. Used as rewards by the protocol and serves as the currency for purchasing in-game items  | Rewards players for just playing the game in SLP tokens (Smooth Love Potion Tokens) as a result players strictly play for the monetary reward  |
| In-game items  | Players use the AI Arena's staple NRN tokens to fund their game item purchases  | Players use the AXS token to fund their game item purchases  |
| Rebuild & Powerups  | Each fighter/player can be depleted and will utilize battle energy known as "voltage" or "voltage points" to initiate a battle. These voltages must be refilled when depleted or 24hrs after the last refill  | No such mechanism exists within the protocol. Essentially, it's become like a game where players are seen as immortals which isn't that much exciting to foster competitiveness  |
| Benefits for playing  | Keep a great track record, then you get to earn NRN tokens and advance your leaderboard ranks. Become mediocre and be punished for points and token slashes - all-in-all breeding player advancements  | Essentially, any one can benefit from earning irrespective of whether or not you're good at the game  |
| Accessibility & Cost  | Will run on Arbitrum One, thereby fostering great adoption due to low cost transactions  | More costly to play and pay transaction fees being a result of being built on Ethereum  |
| AI trained models  | Players can train and set their trained models for the fighters they own. This is done offchain and recorded onchain.  | No such similar model exist  |

**What is unique?**
- Voltage refills - Duels consume energy, 10% of a fighter NFT's voltage to be precise per initiated duel. There isn't such a unique feature yet in other GameFi protocols. These voltages must be refilled once depleted in order to continue initiating and participating in duels.

- Mintpasses - Before the launch of the AI Arena protocol, there are some users who have access to mint the first few fighter NFTs. Once the protocol launches, these users then redeem the mint passes for fighter NFTs.

- Staked risk - Players can typically earn NRN tokens if they win and have some stake (NRN tokens) in the just completed battle. These stakes are facilitated as NRN tokens and earned as long as the player wins and keeps a consistent winning record.


## 7. Architecture Recommendations

**What could be incorporated?**
- Allow the community to create in-game items. Currently, in-game items can only be created by privileged admins. 
- Implementing some kind of tokenization that allows fighter NFTs to be used as collateral for loans in a protocol would be a good addition.
- Payments are currently only made in NRN tokens within the AI Arena protocol. This is great and will foster utility for NRN tokens. It would be better for the team to keep it this way and not allow other payment tokens such as ETH on Arbitrum One.

## **Other recommendations**

### 1. Decentralized model approach
The game is meant to foster player vs player duels and rewarding good track record overtime. One of the main drivers so far for blockchain games have the been the rewards promised to winners/players but another driving factor players pay more attention to are in-game items.

At this stage of the protocol's development it is good to keep the creation of in-game items to a small body of trusted individuals such as the protocol owner or an admin. But as the game advances, players will push for more decentralization of this specific function. They would want to have more flexibility for who can decide/create in-game items that will exist within the protocol.

At some point in the future, if not so soon; players will want to have the ability to update their stakes for a fighter NFT and having this feature be restricted to one of the protocol's other contract will breed this push from players. On the one hand, it makes sense that whatever NRN tokens you have staked on the outcome of a duel for your fighter cannot be pulled out/unstaked mid-game. On the other hand, it also makes sense that I might want to quit a game before it even starts for unforeseen reasons and being unable to undo my stakes or participation is a bummer. 


### 2. The MergingPool contract is vulnerable
This contract has a certain number of vulnerabilites posed but we will try to give an overview of one such vulnerability being the ability for a winner to claim more than their predetermined NFT count to be minted to them.

This is no good. As one transaction breeds a movement and any movement can be seen onchain. With one address running a malicious behavior, it quickly becomes two, three, twenty address doing the same thing. The contract needs to re-think how it handles these edge case problems that breed this movement and render blockades to stop it from ensuing in the first place.


### 3. External calls
There's a relatively low amount of external calls that happen within the protocol.

These calls happen when a player gets minted a player NFT either through claiming fighters, redeeming mint passes for fighter NFTs, and minting from merging pool claims. These are ERC721 token mints which implement the external call check `onERC721Received` to determine if the receiving address can receive/process NFTs. Another contract where such external calls happen is the `GameItems` contract which calls the ERC1155 `_doSafeTransferAcceptanceCheck` on the receiving address to determine if it can process such transferred NFT token.

These external calls aren't vulnerable by themselves when implemented correctly. However, we have been able to exploit a case(s) where a malicious user can gain additional tokens from one such call of which we have suggested a fix for in the corresponding report case.


**Note:**
Consider the following points to judge:

1. A thorough breakdown of each smart contract in the protocol to ensure proper understanding for any reader that comes across this.
2. Representing the smart contract workflow through a visual diagram.
2. Extensive analysis of the contract by function-to-function call trace analysis.
3. Comparison table against another competitive protocol.
4. Best recommendations that can be safely implemented. 


### Time spent:
48 hours (High & Medium findings, QA findings and Analysis report)

### Time spent:
48 hours