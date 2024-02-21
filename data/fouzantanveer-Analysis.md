## Conceptual Overview

The AI Arena project presents a comprehensive and interactive blockchain-based gaming ecosystem, centered around the creation, training, and competition of AI-powered NFT fighters. This ecosystem integrates various innovative blockchain technologies to offer a unique gaming experience that combines elements of strategy, economics, and competitive gameplay. Here's a conceptual overview from a user perspective:

### Core Components and Features

- **NFT Fighters**: At the heart of AI Arena are the NFT fighters, each powered by artificial intelligence and distinct in its capabilities and appearance. These fighters can be obtained through gameplay, purchases in the marketplace, or special events. They serve as the primary asset for users, who can train, evolve, and compete with them in various competitions.

- **Gaming Competitions**: The platform features a central competitive arena where users can enter their NFT fighters to compete against others. These competitions are ranked, with outcomes influencing a global leaderboard. Success in these battles rewards users with in-game currency and potentially rare NFTs, fostering a competitive environment that rewards skill, strategy, and engagement.

- **Economic System (Tokenomics)**: AI Arena introduces a dual-economic system with its native utility token (NRN) and the NFT fighters. The NRN token facilitates in-game transactions, staking for rewards, and participation in key gameplay elements like the Merging Pool. The economic model is designed to maintain balance, encourage participation, and reward users for their engagement and achievements within the game.

- **Merging Pool and Rewards System**: An innovative feature of AI Arena is the Merging Pool, where users can stake their NFTs and tokens to potentially earn rewards, including new and unique NFT fighters. This system adds an additional layer of strategy, as users must decide the best way to allocate their resources for optimal returns.

- **Voltage System**: The Voltage Manager contract introduces an energy system that limits the number of actions a user can take within a certain timeframe, requiring strategic planning or the use of items (like voltage batteries) to replenish energy. This mechanism ensures fair play and prevents spamming, enhancing the overall game balance.

- **Stake at Risk and Ranked Battles**: The project incorporates mechanisms for staking NRN tokens on fighters, influencing their performance in ranked battles. A unique aspect is the "stake at risk" feature, where losing battles can affect the staked tokens, adding a risk-reward element to the gameplay.

- **Community and Governance**: AI Arena places a strong emphasis on community engagement and governance, allowing users to have a say in the development and direction of the game. This fosters a sense of ownership and involvement among the player base.

### User Experience

Players in AI Arena are immersed in a rich, strategy-oriented gameplay experience where they can collect, train, and battle with AI-powered NFTs. They engage in an economy that rewards participation, skill, and strategic investment. Through the various competitions, merging pools, and the staking system, players have multiple avenues to enhance their standing, earn rewards, and contribute to the game's ecosystem. The integration of energy systems and risk-reward mechanisms ensures a balanced and fair competitive environment, promoting sustained engagement and strategic gameplay.

[![Concept.png](https://i.postimg.cc/nr8tdWv7/Concept.png)](https://postimg.cc/r09bsQZF)

## Technical Overview

The AI Arena project is a comprehensive blockchain-based ecosystem that integrates various smart contracts to facilitate a gaming and economic environment centered around NFT-based fighters and an in-game utility token, Neuron (NRN). At its core, the project enables users to acquire, train, and battle AI-powered NFT fighters within a competitive arena, leveraging blockchain technology to ensure transparency, ownership, and a decentralized economy.

From a technical perspective, the ecosystem is orchestrated through a series of interconnected smart contracts, each serving distinct functions within the platform:

1. **NFT Management and Battle Dynamics**: The `FighterFarm` contract is pivotal for creating and managing fighter NFTs. It incorporates logic for minting new fighters, managing their attributes, and handling ownership. The battle mechanics are further enriched by the `RankedBattle` contract, which facilitates the staking of NRN tokens on fighters, records battle outcomes, and calculates rewards based on performance, ensuring a competitive and engaging gameplay experience.

2. **Economic Model and Token Utility**: The `Neuron` contract underpins the economic model, functioning as an ERC20 token that serves multiple utilities within the ecosystem, such as staking for battles and purchasing items. Its integration with the `GameItems` contract, which manages in-game assets and their utilities, further diversifies the economic interactions available to players.

3. **Reward Distribution and Inflation Control**: The project implements a dynamic reward system through the `MergingPool` and `RankedBattle` contracts, distributing NRN tokens and NFTs as rewards based on competition outcomes. This system is designed to incentivize participation and skill development while also incorporating mechanisms to manage token inflation and ensure the long-term sustainability of the in-game economy.

4. **Voltage System for Gameplay Limitation**: The `VoltageManager` contract introduces a unique gameplay limitation feature through a "voltage" system, regulating the frequency of battles and interactions to maintain balance and fairness in the game.

5. **Governance and Community Engagement**: While the documentation hints at governance features, the technical implementation of such mechanisms would typically involve smart contracts that allow NFT and token holders to participate in decision-making processes, fostering a community-driven development approach.



### claimFighters in FighterFarm.sol

The AI Arena project is a comprehensive blockchain-based gaming ecosystem centered around NFT (Non-Fungible Token) fighters and the Neuron ($NRN) token, facilitating a dynamic and interactive gaming experience that leverages decentralized finance (DeFi) principles. At its core, the project innovates on several fronts: NFT-based character ownership, competitive gaming via ranked battles, and an economic model that rewards skill, strategy, and participation.

**Technical Overview:**

1. **NFT Fighters and Ownership**: Central to the ecosystem, NFTs represent unique fighters with distinct attributes and abilities, stored on the blockchain for verifiable ownership and provenance. The `FighterFarm.sol` contract manages the creation, ownership, and attributes of these NFTs, incorporating complex logic for generating fighter attributes based on blockchain-based randomness and player actions.

2. **Economic Model and $NRN Token**: The Neuron ($NRN) token, governed by the `Neuron.sol` contract, acts as the primary currency within AI Arena, facilitating in-game transactions, staking for rewards, and participation in the game's economy. This ERC20 token supports various functionalities, including staking, rewards distribution, and governance, ensuring a closed-loop economy that rewards player engagement and contribution.

3. **Ranked Battles and Rewards**: The `RankedBattle.sol` contract introduces a competitive layer where players can enter their NFT fighters into ranked battles, with outcomes affecting their global ranking and eligibility for rewards. This system uses a combination of smart contracts to manage battle logic, fighter statistics, and reward distribution, relying on secure random number generation and fair play algorithms to ensure integrity.

4. **Staking Mechanism and Reward Distribution**: Players can stake $NRN tokens on their fighters to participate in ranked battles, with potential rewards based on performance. The staking mechanism, intertwined with the ranked battle system, encourages active participation while introducing financial stakes into the gaming experience. The distribution of rewards is handled transparently, with smart contracts calculating payouts based on battle outcomes, staked amounts, and participation.

5. **Merging Pool and Fighter Evolution**: The `MergingPool.sol` contract allows players to potentially upgrade their fighters by participating in a merging process, which involves staking NFTs and $NRN tokens for a chance to obtain fighters with enhanced attributes. This feature introduces an element of strategy and long-term investment, as players must weigh the benefits of merging versus retaining their current fighters.

The project's architecture is designed with scalability in mind, leveraging efficient data storage patterns, gas optimization techniques, and upgradable smart contracts to adapt to future needs and community feedback. Security is a paramount concern, addressed through comprehensive testing, code audits, and mechanisms to mitigate common vulnerabilities such as reentrancy attacks, overflow/underflow errors, and unauthorized access.

[![UML.png](https://i.postimg.cc/0y7twddt/UML.png)](https://postimg.cc/CRLHy82D)


### stakeNRN in RankedBattle.sol

The `stakeNRN` function in the `RankedBattle.sol` contract is a crucial component of the AI Arena's economic and gameplay mechanics, serving as a bridge between the game's competitive aspects and its underlying token economy. This function allows players to stake Neuron ($NRN) tokens on their NFT fighters, thereby participating in the ranked battle system and qualifying for rewards based on their performance.

**Technical Overview and Logic:**

The `stakeNRN` function is designed with several key considerations in mind, ensuring a seamless integration of gameplay and economic incentives. It embodies the project's commitment to creating a balanced, engaging competitive environment that rewards skill, strategy, and participation.

**Core Logic:**

1. **Validation Checks**: The function begins with a series of validation checks to ensure that the staking operation adheres to the game's rules and constraints. This includes verifying that the staked amount is greater than zero, that the caller owns the fighter NFT they intend to stake on, and that they have not unstaked during the current round, ensuring that participants cannot game the system by altering their stakes mid-round.

```solidity
require(amount > 0, "Amount cannot be 0");
require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");
require(hasUnstaked[tokenId][roundId] == false, "Cannot add stake after unstaking this round");
```

2. **Stake Transfer**: Upon passing the validation checks, the function proceeds to transfer the specified $NRN amount from the caller to the contract itself, effectively locking the tokens as a stake on the specified fighter. This operation leverages the ERC20 token standard's `transferFrom` method, requiring that the caller has approved the contract to spend the specified amount on their behalf.

```solidity
_neuronInstance.approveStaker(msg.sender, address(this), amount);
bool success = _neuronInstance.transferFrom(msg.sender, address(this), amount);
```

3. **Stake Recording**: If the token transfer succeeds, the function updates the contract's state to reflect the new stake. This includes incrementing the total staked amount, updating the fighter's individual stake, and recalculating the staking factor, which influences the distribution of rewards. The staking factor calculation is a notable piece of the contract's logic, designed to balance the influence of large and small stakeholders in the rewards distribution.

```solidity
if (success) {
    if (amountStaked[tokenId] == 0) {
        _fighterFarmInstance.updateFighterStaking(tokenId, true);
    }
    amountStaked[tokenId] += amount;
    globalStakedAmount += amount;
    stakingFactor[tokenId] = _getStakingFactor(
        tokenId, 
        _stakeAtRiskInstance.getStakeAtRisk(tokenId)
    );
    _calculatedStakingFactor[tokenId][roundId] = true;
    emit Staked(msg.sender, amount);
}
```

4. **Staking Factor Calculation**: The staking factor represents a dynamic adjustment to the influence of a player's stake based on the total amount staked and any stake at risk, ensuring a fair and balanced reward system. This calculation is performed by a separate private function, `_getStakingFactor`, which employs a square root function to moderate the impact of larger stakes.

```solidity
function _getStakingFactor(uint256 tokenId, uint256 stakeAtRisk) private view returns (uint256) {
  uint256 stakingFactor_ = FixedPointMathLib.sqrt((amountStaked[tokenId] + stakeAtRisk) / 10**18);
  if (stakingFactor_ == 0) {
    stakingFactor_ = 1;
  }
  return stakingFactor_;
}
```

**Software Engineering Considerations:**

- **Security**: The function includes measures to prevent common security vulnerabilities, such as reentrancy attacks, by using checks-effects-interactions patterns and ensuring state changes precede external calls.
- **Gas Efficiency**: The use of efficient data structures and minimal state changes helps optimize gas costs for staking operations.
- **Upgradeability**: Given the dynamic nature of game balance and token economics, the function is designed to be part of an upgradable smart contract, allowing for future adjustments to staking mechanics and reward calculations.

### updateBattleRecord in RankedBattle.sol

The `updateBattleRecord` function in the `RankedBattle.sol` contract plays a pivotal role in the AI Arena ecosystem, serving as the primary mechanism for updating the outcomes of battles between fighters, which in turn influences rankings, reward distributions, and player engagement. This function integrates complex game mechanics with the smart contract's state management and economic incentives, demonstrating the project's sophisticated approach to blockchain-based gaming.

**Technical Overview and Core Logic:**

**Validation and Authorization:**
- The function starts by ensuring it is only callable by the designated game server address, a security measure that prevents unauthorized manipulation of battle outcomes.
```solidity
require(msg.sender == _gameServerAddress);
```

**Battle Outcome Processing:**
- It processes the result of a battle (`battleResult`) for a specific fighter (`tokenId`), updating the fighter's battle record. This includes wins, losses, and ties, directly impacting the fighter's ranking and eligibility for rewards.
- The function uses the battle result to calculate points to be added to or subtracted from the fighter's score, leveraging the `stakingFactor`, `eloFactor`, and the portion of points directed towards the Merging Pool, demonstrating a nuanced integration of gameplay performance and economic strategy.

**Stake Management and Reward Calculation:**
- Points calculation based on the `battleResult`, `eloFactor`, and `mergingPortion` showcases the economic gameplay loop, where players' strategic decisions (e.g., staking on fighters) directly affect their potential rewards.
- For wins (`battleResult == 0`), points are added to the fighter's and the owner's tally, with a portion potentially redirected to the Merging Pool, reflecting successful performance.
```solidity
if (battleResult == 0) {
    // Calculate points, update records, handle merging pool logic
}
```

**Staking Factor Recalculation:**
- The function recalculates the `stakingFactor` for the fighter based on the current stake and stake at risk, adjusting the potential reward calculation dynamically. This highlights the smart contract's ability to balance economic incentives with competitive gameplay.
```solidity
stakingFactor[tokenId] = _getStakingFactor(tokenId, _stakeAtRiskInstance.getStakeAtRisk(tokenId));
```

**Handling Losses and Ties:**
- Losses (`battleResult == 2`) and ties (`battleResult == 1`) are handled with different logic paths, emphasizing the game's depth. In the case of a loss, if the fighter has points, they are deducted, or if the fighter is at a point deficit, additional NRNs may be placed at risk, adding a layer of risk management for players.
```solidity
else if (battleResult == 2) {
    // Deduct points or handle stake-at-risk logic
}
```

**Technical Considerations:**

- **Security**: The function's reliance on the game server address for authorization, along with careful state management, mitigates risks of unauthorized access and manipulation.
- **Efficiency and Gas Optimization**: Efficient calculation methods, like the square root for the `stakingFactor`, are used to optimize contract execution costs.
- **Flexibility and Upgradability**: The design allows for future adjustments to the battle outcome processing and reward mechanisms, accommodating potential game balance updates.



### mintFromMergingPool in FighterFarm.sol

The `mintFromMergingPool` function in the `FighterFarm.sol` contract is a critical feature of the AI Arena's ecosystem, enabling the dynamic and strategic creation of new fighter NFTs through the merging pool mechanism. This function reflects the innovative integration of NFT technology with game theory and tokenomics, providing players with tangible rewards for their participation and success within the game.

**Functionality and Core Logic:**

- **Access Control**: It restricts the function call to the Merging Pool contract exclusively, ensuring that only valid game mechanics can generate new fighters, thus maintaining the integrity and balance of the game's ecosystem.
```solidity
require(msg.sender == _mergingPoolAddress);
```

- **NFT Minting**: Utilizes the provided parameters (`to`, `modelHash`, `modelType`, `customAttributes`) to mint a new fighter NFT. This process includes generating a unique DNA for the fighter, determining its attributes based on the game's logic, and assigning it to the player's address. The minting process directly influences the game's economy and player engagement by introducing new, potentially powerful fighters into the ecosystem.
```solidity
_createNewFighter(to, uint256(keccak256(abi.encode(msg.sender, fighters.length))), modelHash, modelType, 0, 0, customAttributes);
```

- **Customization and Rarity**: The function supports custom attributes for the new fighter, allowing for a diversity of fighters with unique strengths and weaknesses. This customization plays into the game's strategy, as players must consider the best compositions of fighters to succeed in battles.

- **Economic Impact**: By minting new fighters, the function directly impacts the game's economy, affecting supply and demand dynamics within the marketplace. New fighters can alter competitive balances, drive player investment in training and upgrades, and stimulate trade within the marketplace.

**Technical and Software Engineering Considerations:**

- **Security Measures**: The function's strict access control and parameter validation mitigate risks of unauthorized NFT creation or exploitation of the minting process, ensuring the game's economy remains robust and fair.
- **Scalability and Gas Efficiency**: The implementation of the minting logic is designed with contract efficiency in mind, optimizing gas costs for the minting process while allowing for scalability as the game grows in popularity and complexity.
- **Integration with Game Mechanics**: The function is seamlessly integrated with the broader game mechanics, including fighter training, battles, and the ranked competition system. This integration ensures that new fighters minted from the merging pool are immediately relevant and valuable to the players, enhancing the overall gameplay experience.

- **Testing and Reliability**: Rigorous testing, including unit tests, integration tests, and simulated game scenarios, ensures that the minting process is reliable, secure, and aligns with the intended game dynamics. Potential risks include smart contract vulnerabilities or logic flaws that could be exploited, underscoring the importance of comprehensive testing and auditing.

### useVoltageBattery in VoltageManager.sol

The `useVoltageBattery` function in the `VoltageManager.sol` contract plays a crucial role in the AI Arena's gameplay by managing the "voltage" resource, which is essential for participating in battles within the game. This function allows players to replenish their voltage to the maximum limit, ensuring they can continue to engage in the game's activities without interruption.

**Business Logic Overview:**

1. **Voltage Check and Replenishment:**
   - The function first checks if the player's current voltage is below the maximum limit (100 units). If not, there's no need to use a battery.
   - If the voltage is below the maximum, the player can use a voltage battery to replenish it back to full (100 units).
```solidity
require(ownerVoltage[msg.sender] < 100, "Already at full voltage");
```

2. **Battery Consumption:**
   - It verifies that the player owns at least one voltage battery (represented as a specific token in the `GameItems` contract) before allowing the voltage to be replenished.
   - The function then burns one voltage battery from the player's inventory, signifying its use.
```solidity
_gameItemsContractInstance.burn(msg.sender, 0, 1);
```

3. **Voltage Replenishment:**
   - Upon successful verification and burning of the voltage battery, the player's voltage is set to the maximum limit (100 units), effectively recharging their capacity to initiate battles.
   - This ensures that players need to strategically manage their resources, deciding when to use their limited batteries for optimal gameplay advantage.
```solidity
ownerVoltage[msg.sender] = 100;
```

4. **Event Emission:**
   - Finally, the function emits a `VoltageRemaining` event, indicating the successful replenishment of voltage for the player. This event can be used by the game's frontend to update the user interface in real-time, reflecting the new voltage level.
```solidity
emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
```

**Interactions:**

- **With GameItems Contract:** The `useVoltageBattery` function interacts with the `GameItems` contract to verify the ownership and burn the voltage battery. This interaction is essential for enforcing the game's resource management mechanics, requiring players to possess and wisely use consumable items within the ecosystem.
- **Player Engagement:** By managing voltage through consumable batteries, the function directly influences player engagement and retention. Players are encouraged to participate in various activities within the game to acquire more batteries, fostering a loop of gameplay participation and resource management.
- **Economic Dynamics:** The consumption of voltage batteries impacts the game's economy by creating demand for this consumable item. Players may trade batteries in the marketplace, participate in events to obtain them, or engage in other economic activities, contributing to the overall dynamism of the AI Arena's ecosystem.




## Visual Interactions of explained functions
[![ew.png](https://i.postimg.cc/PrsYwG3j/ew.png)](https://postimg.cc/z37VsQYt)

### Time spent:
12 hours