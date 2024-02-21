

# AI Arena Audit Analysis
![logosAIarena](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/fa71ee21-133c-4998-8b44-74a97337ead9)

AI Arena is a PvP battle game on the web3 platform using L2, where the fighters are artificial intelligence (AI) trained by humans. Each fighter has physical attributes, generation, weight, element, fighter type, and model data.  
  

# Overview

  

Structuring the code is crucial for conducting a thorough analysis of the code base. This enables auditors to predict the level of contract difficulty, identify critical contracts, and determine security-critical features such as payment functions or assembly usage. Additionally, this approach accurately measures the time required to perform a detailed audit and helps determine the cost of a project audit. To achieve efficiency, clarity, and improvements, it is essential to have a comprehensive view of the entire architecture. This enables the identification of the basic structure, relationships between modules, and key design patterns. By understanding the big picture, we can analyze the complexity of the code and reveal its strengths and potential for improvement with confidence.


# Approach to Auditing

  

- We began by thoroughly reviewing and read docs AI Arena procotol the provided documentations, as well as reviewing the video provided. Here, we made note of important information, noted down unclear parts and overall tried to fully understand how the protocol should function. We also took notes of integrated and inherited protocols and mapped out possible risk areas.

  

- While we were studying the docs, we had ran static analyzers and linters through the contracts to detect basic vulnerabilities and other non critical errors. The result of our static analysis was then compared to the provided bot reports and those deemed known were removed.

  

- After the documentation review and static analysis, we started the process of manual code inspection. We noted how the contracts were divided into different modules and we inspected the contracts in each module carefully. We stuidied each of the contracts, tested how each function behaves and compared that to how they're expected to behave. We then tried out various attack ideas, including the ones provided by the sponsors. We noted the dependencies, inheritancies, and possible vulnerabilities that can arise from them. We made comparisons to issues found in protocols of the same kind (including older commits) to see if the same mistakes were repeated and how effective the provided fixes were.

  

- After this was done, we made notes on the issues we found and prepared the analysis report.

  
  

# Codebase quality

| Issue                    | Description                                                                                              | Recommendation                                                                                                                                            |
|--------------------------|----------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| Potential reentrancy attack | The reRoll function calls an external contract's function (_neuronInstance.transferFrom) which can potentially allow a reentrancy attack. | Consider using the Checks-Effects-Interactions pattern to prevent reentrancy attacks. Move the external call to the end of the function, after all state changes have been made. |
| Function complexity      | The claimFighters function has multiple responsibilities, including verifying the signature, updating the nftsClaimed mapping, and creating new fighters. | Split the function into smaller, more manageable functions to improve readability and maintainability.                                                |
| Unclear function behavior | The behavior of the _createFighterBase function is not clear, especially how the dna parameter is used to calculate the element and weight values. | Add comments or documentation to explain the function's behavior and how the input parameters are used.                                                 |
| Function complexity      | The _createFighterBase function contains multiple calculations that could be simplified.                    | Simplify the function by calculating the element and weight values in a single line, and removing the unnecessary newDna calculation.                     |
| Memory allocation    | Memory allocation concern in _updateRecord function.                    | The _updateRecord function may lead to memory fragmentation or excessive memory usage due to frequent updates to the fighterBattleRecord mapping. Suggest considering alternative data structures or memory management techniques to mitigate this concern.                     |
| 	Resource leakage     | possibility in _updateRecord function. | The _updateRecord function might not release or reset resources properly after execution, potentially leading to memory leaks. Recommend adding checks or cleanup routines to ensure proper resource management.                     |



## Code Review Suggestions

### Inefficient loop

- **Issue:** Inefficient loop
- **Description:** The loop in the mint function iterates through all game items to find the one with the given `tokenId`. This can be inefficient for large collections.
- **Suggestion:** Replace the array `allGameItemAttributes` with a mapping that uses the token ID as the key.


### Magic number

- **Issue:** Magic number
- **Description:** The number `price` is used without explanation.
- **Suggestion:** Define `const uint256 PRICE_MULTIPLIER = 1e18;` and use it to calculate the price: `uint256 totalCost = allGameItemAttributes[tokenId].itemPrice.mul(quantity).mul(PRICE_MULTIPLIER);`.

### Unchecked arithmetic operation

- **Issue:** Unchecked arithmetic operation
- **Description:** The subtraction `allGameItemAttributes[tokenId].itemsRemaining - quantity` can result in underflow if `quantity` is larger than the remaining items.
- **Suggestion:** Use OpenZeppelin's SafeMath library to perform safe arithmetic operations.


### Magic number

- **Issue:** Magic number
- **Description:** The number 1 is used without explanation.
- **Suggestion:** Define `const uint256 DAY = 1 days;` and use it to calculate the replenish time: `dailyAllowanceReplenishTime[owner][tokenId] = block.timestamp + DAY;`.

### Unchecked arithmetic operation

- **Issue:** Unchecked arithmetic operation
- **Description:** The subtraction `remaining - amount` can result in underflow if `amount` is larger than the remaining allowance.
- **Suggestion:** Use OpenZeppelin's SafeMath library to perform safe arithmetic operations.

### Consider Unclear variable names:

Change it to something more descriptive

- "nrnToReclaim":  such as "amountOfNRNToReclaim."
- "nrnToPlaceAtRisk":  such as "amountOfNRNToPlaceAtRisk."
- "fighterId":  such as "idOfFighter."
- "fighterOwner":  such as "ownerOfFighter."
- "success":  such as "transferSuccessful."
- "price" :  such as `price` to `totalCost`


### Consider  Unclear function purposes:

Function purpose is not clear from its name , Change it to something more descriptive

- "setNewRound": such as "setNewRoundAndSweepLostStake."
- "reclaimNRN": such as "reclaimNRNFromStakeAtRisk."
- "updateAtRiskRecords": such as "updateStakeAtRiskRecordsForFighter."
- "_createFighterBase": such as "_calculateFighterAttributes."                                      


### Strengths :

Exemplary unit test coverage: The codebase shines with its meticulous unit testing strategy. The comprehensive suite of unit tests ensures the reliability and resilience of the system's core functionality, providing confidence in its performance under various scenarios.


Artful integration of Natspec: A commendable strength is the thoughtful integration of Natspec. The use of this documentation format not only improves code readability, but also exemplifies a commitment to providing clear and human-friendly documentation that fosters a more collaborative and accessible development environment.


### Weaknesses :

Opportunities to improve documentation: While the codebase stands out in many ways, there is an opportunity for refinement in the documentation. Addressing this area of improvement can increase the overall clarity of the codebase, make it more accessible, and facilitate seamless collaboration among developers. In essence, the codebase demonstrates proficiency in testing methodologies and documentation practices. Strengthening the documentation further would strengthen the codebase's comprehensibility and contribute to an even more polished and developer-friendly ecosystem.


# Approach we followed when reviewing the code

## Fighter Attributes:

- Physical attributes determine visual appearance.
- Generation influences visual appearance.
- Weight determines combat attributes.
- Element determines special abilities.
- Fighter type indicates whether it's a regular Champion or Dendroid.
- Model data consists of model type and model hash.

## Reward System:

- Players can enter their NFT fighters into ranked battles to earn rewards in the native token $NRN.
- ERC20 token defined in the Neuron.sol smart contract.
- The roles of MINTER and STAKER are given to the RankedBattle.sol smart contract to facilitate the reward system.
- FighterFarm.sol and GameItems.sol smart contracts are given the SPENDER role for in-game purchases with the native token.

## Reward Mechanism:

- Players can only earn $NRN by staking their tokens and winning.
- To equalize the odds, atake the square root of the amount staked to calculate the stakingFactor used in point calculation after each ranked match.

## Voltage Management:

- Each wallet has voltage that needs to be managed. Every 24 hours, the voltage is replenished to 100.
- Each ranked battle requires 10 units of voltage. If a player runs out of voltage, they must wait for it to naturally replenish or they can buy batteries from the GameItems.sol smart contract.

## Ranked Battles:

- Ranked battles allow players from all over the world to enter their NFTs into battles.
- The goal is to climb the global rankings and earn rewards in native Neurons or $NRN tokens.

## Game's Uniqueness:

- AI Arena is the first game where humans are directly involved in the process of improving AI.
- Players can teach AI skills through imitation learning and analyzing its behavior.

## AI-Based NFTs:

- Each NFT is backed by artificial intelligence with skin, skeleton, and core layers.
- Each has different combat attributes, elemental powers, and types.

## AI Training:

- Training is the process of adjusting parameters within the neural network to make the AI act in certain ways in specific situations.
- Training is done through imitation learning with 4 steps: Data Collection, Configuration, Training, and Inspection.

## Entering the Arena:

- Once the AI is ready, players can enter it into the arena.
- The main battle mode is Ranked Battles with an Elo ranking system.

## Ranking System:

- The ranking system is based on the Elo rating mechanism.
- Opponent selection is done randomly from a pool of contestants with similar rating levels.

## Rewards and Risks:

- NRN rewards are based on NFT performance, amount staked, and Elo rating.
- There is a risk of losing points and bets for players who lose.

  

# Analysis of the code base and diagram architecture

  
## AiArenaHelper
![AiArenaHelper sol](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/25a60122-1979-4853-a131-82099e8df9bd)

## FighterFarm
![FighterFarm sol](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/2ee51d34-1501-4a9c-86c2-e7c26838fe7f)

## FighterOps
![FighterOps sol](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/5c50c81f-6184-4179-9c18-90fedeb0a278)

## GameItems
![GameItems sol](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/bceab7b7-7594-47f5-a5b5-c2133de46bc1)

## MergingPool
![MergingPool sol](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/8df4441f-658f-47ac-a9ab-3f8f86e3013d)

## Neuron
![Neuron sol](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/c026fc84-199a-4e89-9d34-fc26bcde59fe)

## RankedBattle
![RankedBattle](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/a4b5625f-957b-469e-a161-a035e91949df)

## StakeAtRisk
![StakeAtRisk](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/8e37de29-d21c-476f-877d-56e2bcfaeda1)

## VoltageManager
![VoltageManager](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/91f0c62d-03a9-4607-a67a-3db2b1d541b5)



# Conclusion

In general, the AI Arena project exhibits an interesting and well-developed architecture we believe the team has done a good job regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from potential malicious use cases. Additionally, it is recommended to improve the documentation and comments in the code to enhance understanding and collaboration among developers. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.


### Time spent:
53 hours