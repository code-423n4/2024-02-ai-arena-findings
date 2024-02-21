# Summary

![.](https://photos.app.goo.gl/Z845Raayvd1hymET8)

**Overview** 

AI Arena is a PvP platform fighting game where the fighters are AIs that were trained by humans. In the web3 version of our game, these fighters are tokenized via the FighterFarm.sol smart contract. Players are able to enter their NFT fighters into ranked battle to earn rewards in our native token $NRN. Our token is an ERC20 token, as defined in the Neuron.sol smart contract. During deployment, we grant our RankedBattle.sol smart contract the MINTER and STAKER roles in order to facilitate our reward system. Additionally, the FighterFarm.sol and GameItems.sol smart contracts are granted the SPENDER role to allow for in-game purchases with our native token. Players are only able to earn $NRN in our game by staking their tokens and winning. However, it is important to note that it is possible for players to lose part of their stake if they perform poorly. Additionally, to level the playing field, we take the square root of the amount staked to calculate the stakingFactor , which is used in the points calculation after each ranked match.

**Scope**

> The engagement involved analyzing nine (9) different Solidity Smart Contracts:
 
- AiArenaHelper.sol: This contract creates mint passes for those who have qualified, which are claimable for AI Arena fighters at a later date

- FighterFarm.sol: This contract manages the creation, ownership, and redemption of AI Arena Fighter NFTs, including the ability to mint new NFTs from a merging pool or through the redemption of mint passes.

- FighterOps.sol: This library defines the Fighter struct and contains methods for fetching information about a fighter.

- GameItems.sol: This contract represents a collection of game items used in AI Arena.

- MergingPool.sol: This contract allows users to stake their fighters to earn rewards.

- Neuron.sol: The Neuron token is used for various functions within the platform, including staking, governance, and rewards.

- RankedBattle.sol: This contract provides functionality for staking NRN tokens on fighters, tracking battle records, calculating and distributing rewards based on battle outcomes and staked amounts, and allowing claiming of accumulated rewards.

- StakeAtRisk.sol: This contract allows the RankedBattle contract to manage the staking of NRN tokens at risk during battles.

- VoltageManager.sol: This contract allows the management of voltage for game items and provides functions for using and replenishing voltage.

# Codebase quality analysis 

**Unique:* 
- Codebase carries out specific governance mechanisms that are uniquely designed for AI Arena’s specific use case e.g Players are able to enter their NFT fighters into ranked battle to earn rewards in native token $NRN.

**Existing Patterns:* 
- AI Arena adheres to common contract management patterns, such as the use of ```solidity AccessControl```.

**Strengths:*
- Named imports of parent contracts are well utilised. 
- Contracts use clear headers to easily identify various code sections.

**Weaknesses:*
- All contracts in scope use floating solidity pragma versions this is not recomended.
- Incomplete test coverage (90%). 100% test coverage is recomended 

# Mechanism review

- The mechanisms implemented in the AI Arena ecosystem, including the ERC721 , standard, ERC1155 standards, and the various modules for ownership management, execution and extension, are innovative and well-designed. They provide a comprehensive range of features and capabilities, enabling a wide range of use cases for NFTs.
 
# Centralization risks
 
Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

File: src/Neuron.sol

```solidity
11: contract Neuron is ERC20, AccessControl { 
```

# Systemic risks
 
- External Contract Dependencies: AI Arena relies on external contracts from Openzepplin. If any of these contracts have vulnerabilities, it would affect the protocol.

- Test Coverage: The test coverage provided by AI Arena is: 90% however, 100% tests coverage is recommended.

- Like any smart contract-based system, AI Arena is exposed to potential coding bugs or vulnerabilities. Exploiting these issues could result in the loss of funds or manipulation of the protocol. 

# Documents 

The AI Arena documentation is quite comprehensive and detailed, providing a solid overview of how the protocol is structured and how its various aspects function. However, there is room for additional details, such as more diagrams, to gain a deeper understanding of how different contracts interact and their functions. 
I would also recommend creating quality Medium articles, it’s a great way to provide an in-depth look at many of the topics in the project and it is used by many blockchain projects.

# Automated Findings 

See automated findings [here](https://github.com/code-423n4/2024-02-ai-arena/blob/main/bot-report.md)

The 4naly3er report can be found [here](https://github.com/code-423n4/2024-02-ai-arena/blob/main/4naly3er-report.md).

# Architecture recommendations
 
 > AI Arena’s architecture seems solid in general, none the less here are some areas that could be improved:

- Testing and Simulations: I recommend creating a live testnet app. Here is an
[example](https://app.opendollar.com/) from The Open Dollar protocol. Conduct thorough testing of all contracts and functions and simulations to understand how they will behave under various market conditions.

- Monitoring Recommendations: While audits help in identifying code-level issues in the current implementation and potentially the code deployed in production, the AI Arena team is encouraged to consider incorporating monitoring activities in the production environment. Ongoing monitoring of deployed contracts helps identify potential threats and issues affecting production environments. With the goal of providing a complete security assessment, the monitoring recommendations section raises several actions addressing trust assumptions and out-of-scope components that can benefit from on-chain monitoring.

- I recommend rewriting some of the tests in the codebase for this audit to use the actual contracts instead of mock addresses like in some cases. This will offer greater confidence during system deployment.

- Financial Activity: Consider monitoring the token transfers over the bridge to identify: Transfers during normal operations to establish a baseline of healthy properties. Any large deviation, such as an unexpectedly large withdrawal, may indicate unusual behavior of the contracts or an ongoing attack. Transactions that revert. These may indicate a user interface bug, an ongoing attack or other unexpected edge cases.

**Gas Optimizations**

- Review data types: Analyze the data types used in your smart contracts and
consider if they can be further optimized. For example, changing uint256 to
uint128 or uint94 can save gas and storage slots.
- Struct packing: Look for opportunities to pack structs into fewer storage slots. By carefully selecting appropriate data types for struct members, you can reduce the overall storage usage.
- Use constant values: If certain values in your contracts are constant and do
not change, declare them as constants rather than storing them as state variables. This can significantly save gas costs.
- Avoid unnecessary storage: Examine your code and eliminate any unnecessary storage of variables or addresses that are not required for contract functionality.
- Storage vs. memory usage: When working with arrays or structs, consider whether using storage instead of memory can save gas. Using storage allows direct access to the state variables and avoids unnecessary copying of data.
- Replacing the use of memory with calldata for read-only arguments in external  functions.

**Other recommendations**

- Regular code reviews and adherence to best practices.
- Conduct external audits by security experts.
- Consider open-sourcing the contract for community review.
- Maintain comprehensive security documentation.
- Establish a responsible disclosure policy for vulnerabilities.
- Implement continuous monitoring for unusual activity.
- Educate users about risks and best practices.

# Approach taken in evaluating the codebase
 
> My analysis of the AI Arena protocol Included understanding the architecture, mechanism, overall codebase, and possible risks associated with the protocol.

- Day 1: I spent time reading the available documentation in order to have a deep understanding of the protocol. 

- Day 2: I analyzed the codebase for better understanding, Performed a Mechanism review, and investigated possible systemic risks, and centralization risks. 

- Day 3: I dedicated this day to coming up with possible Architecture recommendations after identifying possible risks and prepared the final analysis report.

**Conclusion**

The AI Arena ecosystem is a well-designed and innovative NFT platform fighting game offering a wide range of features and capabilities. However, there are areas where improvements could be made, particularly in terms of permission management, gas efficiency, and potential exploitation of certain modules. By addressing these issues,  AI Arena could become even more secure, efficient, and reliable.




### Time spent:
24 hours