


# AI Arena contest Analysis:

## Description overview of The AI Arena Contest

When reading the documentation and exploring the AI Arena protocol website, it becomes evident that they have key contracts that play a pivotal role in the protocol's functionality. The AiArenaHelper contract stands as a cornerstone within the protocol, tasked with generating and overseeing the physical attributes of AI Arena fighters. Its role is indispensable, shaping the defining characteristics and capabilities of the in-game combatants.the AiArenaHelper and  FighterFarm contract work to gather FighterFarm, serves as a management system for creating, owning, and redeeming AI Arena Fighter NFTs. It facilitates various functionalities such as minting new fighters from a merging pool, redeeming mint passes for NFTs, updating fighter models, transferring ownership, and more.
When the protocol seeks to manage the collection of game items, it leverages the GameItems contract as a foundational tool.
the contract implements the ERC1155 standard for managing fungible and non-fungible tokens (NFTs). The contract allows users to mint, transfer, and manage game items, while also providing functionality for admins to adjust item attributes and permissions.
As users aim to earn new NFTs, the MergingPool contract diligently tracks the accumulation of points by fighters. It facilitates administrators in selecting round winners, who are subsequently eligible to claim their rewards.
As the battle draws to a close, the RankedBattle contract takes charge, meticulously tracking battle records, crunching numbers to determine rewards, and distributing them in accordance with battle outcomes and staked amounts. Additionally, it enables users to claim their hard-earned rewards accumulated throughout the battles.
At the helm of risk management, the StakeAtRisk contract takes on the responsibility of safeguarding NRN (Neuron) tokens vulnerable to loss throughout competitive rounds. With precision, it monitors and records the stake at risk for individual fighters within each round, facilitating the reclamation of NRN tokens from the brink of loss, and diligently updating stake records as battles unfold. As the curtains fall on each round, it orchestrates the seamless transfer of lost stakes to the treasury contract, ensuring a robust and secure ecosystem for all participants.

## Admin Rules and Abilities:

The AiArenaHelper contract includes a function (transferOwnership) that allows the current owner to transfer ownership to a new address. Only the current owner can invoke this function.
admin can add attribute divisors for attributes through the addAttributeDivisor function. This function allows customization of the DNA divisors for attributes, affecting attribute generation.
admin can add or delete attribute probabilities for different generations using addAttributeProbabilities and deleteAttributeProbabilities functions, respectively.
The StakeAtRisk contract constructor sets the community treasury address and the addresses of the Neuron contract and the RankedBattle contract.
Only the RankedBattle contract is authorized to set a new round for staking and to reclaim NRN tokens from the stake at risk.
The RankedBattle contract can also update the stake at risk records for a fighter.
the admin can increment the generation of a specified fighter type.
but in the GameItems contract  The owner can grant or revoke admin access to other addresses, allowing them to perform certain administrative tasks.
Admins can select winners for the current round and reset their points.

## Approach taken in evaluating the codebase

High-level overview: I analyzed the overall codebase in one iteration to get a high-level understanding of the code structure and functionality.

Documentation review: I studied the documentation(https://docs.aiarena.io/gaming-competition) to understand the purpose of each contract, its functionality, and how it is connected with other contracts and work of the AI Arena protocol.

Literature review: I read old audits and known findings, as well as the bot races findings.

Functionality Analysis: Review the purpose and functionality of each function.

Testing setup: I set up my testing environment and ran the tests to ensure that all tests passed. Used foundry and slither to test this protocol or fuzz as well.

Gas Optimization: Analyze gas usage and suggest optimizations where possible.

Detailed analysis: I started with the detailed analysis of the code base, line by line. I took the necessary notes to ask some questions to the sponsors.

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the AI Arena protocol. These risks encompass concentration risk, admin roles and centralization risks arising.

Here’s an analysis of potential systemic and centralization risks in the contrac

### Systemic Risks:

1. The AiArenaHelper.sol contract relies on an external FighterOps contract for defining the structure of fighter physical attributes (FighterOps.FighterPhysicalAttributes). Any changes or vulnerabilities in this dependency could affect the functionality of AiArenaHelper.
2. The GameItems contract appears well-designed and covers various aspects of game item management. However, improper configuration of admin access or vulnerabilities in external contract integration could pose systemic risks to the overall system.
3. The RankedBattle.sol reliance on external contracts (e.g., FighterFarm, VoltageManager, Neuron) introduces inter-contract dependencies and potential risks if these contracts behave unexpectedly.
The complexity of reward calculations and staking mechanisms could introduce vulnerabilities if not implemented correctly.
4. The StakeAtRisk.sol contract  if the RankedBattle contract malfunctions or behaves unexpectedly, as it plays a crucial role in coordinating with the StakeAtRisk contract.
The reliance on the Neuron contract for transferring NRN tokens introduces a systemic risk if the Neuron contract encounters issues or is compromised
5. The VoltageManager.sol contract if the contract owner abuses their privileges or if admins make biased decisions regarding access to voltage spending.
Furthermore, if the game items contract instance encounters issues or is compromised, it could affect the functionality of voltage replenishment.

### Centralization Risks:

1. the AiArenaHelper.sol conotract Ownership of the contract is initially assigned to the deploying address and can be transferred to another address using the transferOwnership function. However, this mechanism still introduces a degree of centralization, as the contract owner holds significant control over the contract's functionality and data.
2. The FighterFarm.sol contract relies on roles designated by the owner (hasStakerRole). These roles centralize control over certain functions, potentially affecting decentralization.
3. The GameItems contract owner holds significant power, including the ability to transfer ownership and adjust admin access. This centralized control could pose risks if misused or compromised.
4. The MerginPool.sol contract owner holds significant power, including the ability to transfer ownership and adjust admin access. This centralized control could pose risks if misused or compromised.
5. The RankedBattle.sol contract owner initially has significant control over contract functionalities and configurations.
Admins, who are designated by the owner, also hold considerable power to adjust admin access and key contract parameters.
There's a reliance on the game server address for updating battle records, introducing a potential central point of failure or manipulation.
6. The StakeAtRisk.sol contract does not pose significant centralization risks since it primarily facilitates the management of NRN tokens during battles. However, it depends on the RankedBattle contract for setting rounds and reclaiming NRN tokens, which could introduce centralization if the RankedBattle contract is controlled by a single entity.
7. The VoltageManager.sol contract owner initially has significant control over contract functionality, including the ability to transfer ownership and adjust admin access.
Admins can further influence contract behavior by controlling which addresses are allowed to spend voltage.
However, these centralization risks can be mitigated by proper governance mechanisms and transparent decision-making processes.

## Gas Optimization

| Number | Issue | Instances |
|--------|-------|-----------|
|[G-01]| Refactor functions to combine events to reduce gas cost | 1 |
|[G-02]| Avoid Unnecessary Use of Storage | 2 |
|[G-03]| Using assembly to revert with an error message | 22 |
|[G-04]| Don’t cache if used only once | 3 |
|[G-05]| Switching between 1 and 2 instead of 0 and 1 (or false and true) is more gas efficient | 12 |
|[G-06]| aching global variables is more expensive than using the actual variable (use msg.sender instead of caching it) | 1 |
|[G-07]| Avoid Assigning a Value that Will Never be Used | 12 |
|[G-08]| Check Arguments Early | 5 |
|[G-09]| State variables that could be declared constant | 2 |
|[G-10]| Inefficient assignments to state variables | 20 |
|[G-11]| Using Storage Instead of Memory for structs/arrays Saves Gas | 5 |
|[G-12]| Use assembly to write address storage values | 31 |
|[G-13]| Redundant state variable getters | 4 |
|[G-14]| Refactor functions such that all checks are performed before assigning values to state variables | 1 |
|[G-15]| Precompute Off-Chain When Possible | 1 |
|[G-16]| Use uint8 Can Increase Gas Cost | 39 |
|[G-17]| Store Data in calldata Instead of Memory for Certain Function Parameters  | 11 |
|[G-18]| use Unchecked Blocks: | 41 |
|[G-19]| Use the existing Local variable/global variable when equal to a state variable to avoid reading from state | 3 |
|[G-20]| Expensive operation inside a for loop | 4 |
|[G-21]| Use Mappings Instead of Arrays | 4 |
|[G-22]| Do not Calculate constant | 3 |

## Recommendation:

Enhanced Decentralization: Consider implementing a multisig mechanism for critical administrative actions, reducing reliance on a single owner and centralized roles.
External Dependency Management: Implement measures to mitigate risks associated with external dependencies, such as regular audits, version control, and utilizing decentralized oracles.
Security Audits: Conduct comprehensive security audits to identify and mitigate potential vulnerabilities, especially in interactions with external contracts.
Gas Optimization: Optimize gas usage where possible to reduce transaction costs and improve efficiency.
Documentation: Provide detailed documentation for users and developers, including explanations of contract functionality, usage instructions, and interactions with external contracts.
Continuous Monitoring: Implement mechanisms for continuous monitoring and auditing of contract activities to detect and address any anomalies or unauthorized activities promptly.

## Time spent 

45 Hours


### Time spent:
45 hours