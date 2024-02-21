

# Ai Arena Audit Analysis

AI Arena is a platform game where you train an AI character to battle other similar players. Imagine a cross between Pokémon and Super Smash Bros, but the characters are AIs, and you can train them to learn almost any skill in preparation for battle.
Players can stake `NRN` tokens and earn enough points to earn a percentage of the total pooled staked `NRN` tokens based on the amount of points they've earned. Players can also use those points to "bet" their tokens on the mergingPool for a chance to earn another NFT fighter.

# Overview

For a thorough code base analysis, it is essential to understand the entire architecture comprehensively. This understanding provides a clear view of the basic structure, the relationships within, and key design patterns, facilitating the prediction of contract difficulty levels, the identification of critical contracts, and the determination of security-critical features, such as payment functions or assembly usage. Moreover, structuring the code aids in accurately estimating the time needed for a detailed audit and consequently helps in assessing the project audit's cost. Through achieving efficiency, clarity, and improvements, this method not only allows for the analysis of code complexity but also confidently unveils the code's strengths and areas ripe for enhancement.


# Approach to Auditing


My process began with an in-depth examination of the documentation and video materials provided by Ai Arena, during which I highlighted critical information, flagged areas of uncertainty, and endeavored to grasp the intended operation of the protocol comprehensively. Additionally, I identified both integrated and inherited protocols and pinpointed potential risk zones.

During my review of the documents, I used my automated bot over the contracts in scope, finding basic vulnerabilities and high potential ones. This yielded clues regarding possible issues.

Following the examination of the documentation and analysis of the bot's findings, I started a manual inspection of the code. This phase involved a detailed assessment of how the contracts were organized and a meticulous review of each segment. I evaluated the functionality of each contract, testing the behavior of individual functions against their expected outcomes and exploring various attack strategies.

At the end of the process, I compiled my observations on the identified issues and worked on the analysis report.


### Strengths :

Excellent test coverage: The codebase stands out for its rigorous approach to unit testing. A thorough collection of unit tests guarantees the system's core functions are both reliable and robust, showing proper capability to perform consistently across different situations..


### Weaknesses :

While the codebase excels in various aspects, enhancing the documentation presents an opportunity for further refinement. Improving documentation clarity can make the codebase more user-friendly, ensuring easier access and fostering better collaboration among developers. Essentially, the codebase showcases strong capabilities in testing methodologies and documentation standards. By bolstering the documentation, we can elevate the codebase's understandability, thereby creating a more refined and developer-centric environment.

# Short summary for each contract in scope

AiArenaHelper: This contract generates and manages an AI Arena fighters physical attributes.

FighterFarm: This contract manages the creation, ownership, and redemption of AI Arena Fighter NFTs, including the ability to mint new NFTs from a merging pool or through the redemption of mint passes.

FighterOps: This library defines the Fighter struct and contains methods for fetching information about a fighter.

GameItems: This contract represents a collection of game items used in AI Arena.

MergingPool: This contract allows users to potentially earn a new fighter NFT.

Neuron: The Neuron token is used for various functions within the platform, including staking, governance, and rewards.

RankedBattle: This contract provides functionality for staking NRN tokens on fighters, tracking battle records, calculating and distributing rewards based on battle outcomes and staked amounts, and allowing claiming of accumulated rewards.

StakeAtRisk: This contract allows the RankedBattle contract to manage the staking of NRN tokens at risk during battles.

VoltageManager: This contract allows the management of voltage for game items and provides functions for using and replenishing voltage.

--

AiArenaHelper:

- Basically manages an NFT AI Fighter attributes.

FighterFarm :

- Handles the creation and claiming of NFT fighters.
- Allows players to mint fighters using a mint pass.
- Allows players to claim fighters from the merging pool.
- Allows players to transfer their NFT fighters to other players.

FighterOps:

- Handles fighter data and information.

GameItems:

- Handles the game items logic, which is the battery in order to replenish the player's voltage.

MergingPool:

- Handles the merging pool, allowing players to stake their points for a chance to win an NFT fighter.

Neuron:

- ERC20 token of the `NRN` token used for staking in AI Arena.

RankedBattle:

- Handles all the logic for ranked battles when players initiate a fight or receive a fight.
- Staking and unstaking is also handled here allowing players to stake their `NRN` token for potential points.

StakeAtRisk:

- Responsible for the staking logic.

VoltageManager:

- Handles the voltage usage of players during ranked battles.

  

# Analysis of the code base and diagram architecture

Use this, I created a simple diagram for each contract to see if functions has calls to other functions allowing me to get a broader view of the project's logic.

## AiArenaHelper
![aiarena_2024_01-AiArenaHelper sol drawio](https://gist.github.com/assets/23437371/2ea10087-6e59-4db9-9fbc-5fce5271e71d)

## FighterFarm
![aiarena_2024_01-FighterFarm sol drawio](https://gist.github.com/assets/23437371/547b77c4-e110-40b1-9937-e97e6ff4a76c)

## GameItems
![aiarena_2024_01-GameItems sol drawio](https://gist.github.com/assets/23437371/cd3ffc95-2997-4901-b8aa-b4b9a3be4e84)

## MergingPool
![aiarena_2024_01-MergingPool sol drawio](https://gist.github.com/assets/23437371/40997035-1569-4a8c-a94a-1290d4a46ebd)

## Neuron
![aiarena_2024_01-Neuron sol drawio](https://gist.github.com/assets/23437371/88293d24-1028-447e-ba5a-68d7ee501411)

## RankedBattle
![aiarena_2024_01-RankedBattle sol drawio](https://gist.github.com/assets/23437371/09642f3b-917d-4bba-94e7-413f0a36559a)

## StakeAtRisk
![aiarena_2024_01-StakeAtRisk sol drawio](https://gist.github.com/assets/23437371/972b8044-ab85-4101-80ae-e94e0a7f9ddc)

## VoltageManager
![aiarena_2024_01-VoltageManager sol drawio](https://gist.github.com/assets/23437371/7556b177-2806-4ff4-a759-be81ea5a7045)

## RankedBattle Logic
I proceeded to create a general logic flow for Ranked Battle.
![aiarena_2024_01-Logic drawio](https://gist.github.com/assets/23437371/568a4431-b3e3-48c2-843a-4e9542539295)

# Conclusion

The AI Arena showcases an interesting architecture and a great concept, reflecting the team's commendable effort in code development. However, several identified risks necessitate immediate attention to safeguard the protocol against potential misuse. It is also advised to enrich the documentation and code comments, boosting both comprehension and collaborative efforts among developers. Further investment in security practices, including mitigation strategies, thorough audits, and bug bounty programs, is highly recommended to preserve the project's security and dependability.

### Time spent:
~30 hours

### Time spent:
30 hours