# Overview

AI Arena is a game that you train your Fighter NFT with AI to battle others. AI training and gameplay take place off-chain, while on the blockchain, Fighter NFT ownership and stats are managed, ERC1155 form items are managed, NRN tokens are staked, game results are registered to earn points and rewards are distributed.

# Approach taken in evaluating the codebase

| Stage | Detail |
| --- | --- |
| Compile and run test | Clone the repository, install dependencies, set environment and run test. |
| Check scope and sort | List the scopes, sorting them in ascending order. Identify core and peripheral logic. |
| Read docs | Read code4rena README and docs. Research on who’s sponsors and what’s their goal. Review the sponsor's existing projects, if any. |
| Understand core logic | Skim through the code to understand the core logic. |
| Manuel Code Review | Read the code line by line to understand the logic. Find bugs due to mistakes. |
| Organize logic with writing/drawing | Organize business logic and look for logic bugs. |
| Write report, PoC | Write the report and make PoC by using test code. |

# Architecture recommendations

![Architecture](https://gist.github.com/assets/70058709/6f8885cd-7a2e-4656-bcdc-c0b3c8f1ce9d)

This is architecture of the contract. It mainly involves NFT minting and management, staking, game results updates, and game item usage. Major management tasks such as updating the game results, selecting winners, or updating rounds are performed with the administrator EOA account.

The winner of the Merging pool is chosen off-chain and registered by the admin. The winner's information is simply stored in an array, and the user has to inefficiently traverse to check if they've won in order to claim their reward. If the number of winners in the Merging pool is significantly increased, it would be more efficient to use a merkle tree.

# Centralization risks

This project is highly centralized, so special attention needs to be paid to account management. The administrator can change the address of the main contracts they interact with, can change reward related settings, and can mint new Fighter NFTs.

The protocol is closer to storing data on the contract with an EOA account managed by the team rather than providing decentralized services. As each EOA plays an important role, it is recommended to strengthen security by using a multi sig wallet.

The following roles exist:

- owner
- admin
- treasury
- delegated
- game server

## Owner

Initially, the deployer is set as owner. Owner has the right to set admin, initialize the contract, and change the contract's settings.

Functions that initialize important variables such as `instantiateNeuronContract` generally do not need to be called again, but are not blocked from being called again. In other words, the owner has a large authority that can even change the important address variable. To mitigate this, limit the initialization functions to be callable only once.

Also, since it is an account with many authorities, it is better to use a 2 step owner to prevent changing to a wrong address by mistake.

## Admin

It has the authority to change various settings of the contract. It also has the authority to register winners in the MergingPool. It also has the authority to airdrop the NRN. It performs management tasks such as updating round or updating the amount of rewards.

## Treasury

The Treasury account collects the NRN tokens used for item purchases and rerolls. It also collects the NRN lost by the user in the game. Since the airdrop is in the form of approving treasury’s token to users, the treasury can also be considered to have the authority to airdrop NRN.

## Delegated (Signer)

Delegated sets the Fighter tokenURI (metadata setting) and signs for minting Fighter tokens. In other words, they have the authority to mint Fighter tokens.

## Game server

The game server account registers the game results on the contract. The contract fully trusts the Game server account, and because it can distribute rewards based on game point or take away staked tokens, the game server has large authority.

To mitigate the authority of the game server, there is a way to receive the user's signature to upload the game results. If you get a signature from the user who starts the game, you will be able to mitigate the ability of the game server account to freely record game results.

# Mechanism review

## Fighter NFT Minting

There are three ways to mint Fighter NFTs.

1. Use MintPass to request
2. Receive a signature from the Delegated and request
3. Win in MergingPool and request

### Use MintPass to request

![Use MintPass to request](https://gist.github.com/assets/70058709/89014dc3-66b4-4bc7-b888-4143062aaf91)

The AAMintPass contract is an ERC721, an NFT that was deployed before FighterFarm. Users can use (burn) their MintPass NFTs to mint an equal number of Fighter NFTs.

### Receive a signature from the Delegated and request

![Receive a signature from the Delegated and request](https://gist.github.com/assets/70058709/53a94f88-50e6-42f4-b11d-43036cdf8f64)

User can request NFT minting by receiving a signature from the signer. User can mint NFTs in the quantity allowed by the signature.

### Win in MergingPool and request

![Win in MergingPool and request](https://gist.github.com/assets/70058709/62f42734-9e85-4c5b-a509-64edc31c0ea1)

Winners of MergingPool are selected and registered by admin every round. Winners can mint NFTs in the number they won. Users can customize the weight and element.

### DNA and attributes

Fighter NFTs have unique attributes. Except reward of MergingPool, these are pseudo-randomly set by DNA values. There are three elements, and weight is set between 65 and 95. FighterFarm generates element and weight, while AiArenaHelper is responsible for generating the appearance attributes.

![DNA and attributes](https://gist.github.com/assets/70058709/610d00eb-08b5-4e34-8bb3-9074b56673ec)

Users can also reroll attributes by paying NRN tokens. When a reroll is requested, a new DNA is assigned, and the attributes of the NFT are changed to new pseudo-random values. Each NFT has a limit on the number of rerolls, so infinite changes are not possible.



## Staking and rewards

You need to stake NRN tokens to get points when you win. The more you stake and the higher your Elo score, the more points you can get when you win. When the game round ends, you can claim rewards proportional to the points you have. Rewards are provided by minting NRN tokens.

### Stake/Unstake

![Stake Unstake](https://gist.github.com/assets/70058709/42012154-71c7-41ea-ac52-2390ce969bce)

Users can stake or unstake NRN to their own Fighter NFT. If you unstake in this round, you can no longer stake additional. Once staking starts, the relevant Fighter NFT is blocked from transfer, and when it is completely unstaked and there is no more staked amount left, the Fighter NFT is unblocked to enable transfer.

### Losing the game

![Losing the game](https://gist.github.com/assets/70058709/9c8378f4-34af-4f13-b7c2-62635cbc0377)

The game server account registers the game results on-chain. If the user who lost the game has staked, the user's points are reduced, and if there are no more points to reduce, n% of the stake is moved to the StakeAtRisk contract each time they lose.

### Winning the game

![Winning the game](https://gist.github.com/assets/70058709/b0fb5a82-bb51-43ac-874b-9209941db059)

If you won the game, you can retrieve some of the tokens at stake-at-risk. If there are no more stake-at-risk tokens left, you can earn points. If you earn points, some of them are passed to the MergingPool. Points are used as the basis for picking winner in the MergingPool later.

### Moving to the next round

![Moving to the next round](https://gist.github.com/assets/70058709/83e70c65-d387-4ca0-92bc-5c295e062064)

Admin finishs this round and starts the next round. When the round changes, the NRN tokens that went to StakeAtRisk move to the treasury. Since a new round has started, users can no longer recover past round stake-at-risk tokens.

### Claim NRN reward

![Claim NRN reward](https://gist.github.com/assets/70058709/35d0f1f0-9e02-47a8-bcb4-f3a38af3d536)

Users can receive rewards proportional to the points earned in the previous round. Rewards are provided by minting new NRN tokens.

## Voltage and Battery

### Use voltage

![Use voltage](https://gist.github.com/assets/70058709/f0286c9a-2108-466a-bb8c-a88added3d8d)

The user who started the game uses voltage. Each game start consumes 10 voltage, and after consuming voltage, it is charged (reset) to 100 after 24 hours. This is processed when the game server registers the game results.

### Battery

![Battery](https://gist.github.com/assets/70058709/e0ebc3e6-8c28-43ce-a8bf-f3e54cf87399)

Users who have consumed the voltage can use the battery to recharge the voltage. The battery is a game item minted in the form of ERC1155. Used batteries are burned.

![Buy battery](https://gist.github.com/assets/70058709/b6ecc104-1163-4b47-8e18-16475d3b89f4)

The GameItems contract is a contract where users can pay NRN and buy items. Items are minted as ERC1155, and currently, only Battery items exist. In the future, admin can add new items, and users can pay NRN and purchase them. The NRN for purchase is transferred to the Treasury.

Various settings can be set for the item. Admin can limit the number of purchases per day, or limit the total number of sales for limited sales. Through settings, certain items can be made non-transferable, creating non-exchangeable items.

# Systemic Risks

You don't have to be extra cautious about frontrunning because this project uses the Arbitrum chain. However, because the frontend, game server, and blockchain are integrated, special attention needs to be paid to race conditions between off-chain and on-chain. Problems can occur if NFTs move or changes occur in staking while the game is still being processed. Before starting the game, a transaction should be generated to lock the NFT on the blockchain, and it needs to be unlocked when the game ends to prevent race conditions.

Also, it is important to generate NFTs randomly, but currently, all random values are implemented as pseudo-random. All of these are predictable and manipulable values. It is recommended to use Chainlink VRF or at least use random values generated off-chain and verify it with a signature.



### Time spent:
40 hours