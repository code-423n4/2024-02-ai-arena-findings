AI Arena is a PvP web3 fighting game where players train and compete with AI fighters tokenized as NFTs through the `FighterFarm.sol` smart contract. The game allows players to stake their native ERC20 token, `$NRN`, to participate in ranked battles and earn rewards based on their performance. The `Neuron.sol` smart contract manages the token's distribution and staking mechanisms.
Players can improve their AI fighters' skills and abilities through a process called Imitation Learning. The game abstracts the AI research process into a skill-based game loop, allowing players to learn about AI while playing the game.
The AI Arena aims to introduce a native token, Neuron ($NRN), which is an essential utility token for player strategy in the on-chain version of the game. The game's core game logic runs off-chain, with the game server acting as an oracle for ranked match results.
AI Arena Basically is a web3 fighting game that combines blockchain technology, AI research, and eSports competition.

Primary goals are to
1.Provide an engaging and skill-based gaming experience. 
2.AI research and collaboration.
3.Establish a thriving ecosystem around the $NRN token.

Its worthy to note that this audit report does not include repetitive documentation that the team is already aware of,It does include suggestions to provide more clarity on certain aspects in the documentation.
The report is crafted towards providing the sponsors with value such as unknown edge case scenarios, faulty developer assumptions and unnoticed architecture-level weak spots. This analysis it is to provide the judge with more context on a specific high-level or in-depth scenario for ease of understandability.


Approach taken in evaluating the codebase
 Followed my pattern that has proven to detect low hanging fruits in terms of exposing vulnerabilities.

2.1 Audit Documentation and Scope
Understood the idea of the project and grasp a high level mental model of the contract interactions, the Initial step involved, was thoroghly examining `audit documentation and scope` to grasp the audit’s functionalities and boundaries, so as to prioritise my efforts. It is worth highlighting the good quality of the `README` for this audit, as it provided valuable insights and actionable guidance that greatly facilitated the onboarding process.

2.2 General insight on AI Arena gameing Funtionality

A good amount of hour was put in gaining knowledge on how the conventional game works, and this helped me understand fully the problem each contract is trying to solve, the gap its trying to bridge, their goals and how the methods implemented not just work, but to what degree better than the conventional.

2.3 Setup and Testing

Setting up test environment to execute forge test, asked more questions and found ways to still further  test as it greatly enhanced the efficiency of the auditing process. With a fully functional test tap at my disposal, which not only accelerate the testing of intricate concepts and potential vulnerabilities but i also gain insights into the developer’s expectations regarding all implementations. 

2.4 Code review

The code review commenced with understanding the pattern” used to manage governance and functionality accross the system. Thoroughly understanding this pattern made understanding the protocol contracts and its relations much smoother. Throughout this stage, Added inline bookmarks in the code in the form of questions, such as, “Could X do Y to mint more tokens or Z bypass this Z check?” Noted down some obvious gas optimizations and low-severity issues not covered by the bot.I documented observations and raised questions concerning potential exploits without going too deep while uncovering the codes intent.

2.5 Threat Modelling

At this point I began formulating specific assumptions that, if compromised, could pose security risks to the system. This process aids me in determining the most effective exploitation strategies. Thoroughly examining and marking any doubtful or vulnerable areas within the protocol, diving deep into these areas, performing in-depth examinations, and subjecting them to rigorous testing to report these issues if they dont checkout.

2.6 Known Finding 
Read old audits and already known findings. Went through the bot races findings checking what was found so as to not report these issues again, while bearing in mind invariants and critial parts of functions that can lead to users lossing funds.

2.7 Report Issues
I started with auditing the code base indepthly this way I started understanding line by line code and took the necessary notes to ask questions, marking out vulnerabilities and how they can be exploited, This assessment helps in evaluating whether these exploits can be strategically combined to create a more significant impact on the system’s security. In some cases, seemingly minor and moderate issues can compound to form a critical vulnerability when leveraged wisely. This has to be balanced with any risks that users may face.


Architecture:
The AI Arena project utilizes a combination of smart contracts and off-chain game server architecture. The smart contracts are developed using Solidity, and they manage the token economy, NFT fighter management, and staking mechanisms. The off-chain game server acts as an oracle for ranked match results.

Strengths:
1. Integration of blockchain technology: By integrating blockchain technology, the AI Arena project can provide a transparent and tamper-proof gaming ecosystem, which can help build trust among the players and the community.

2. Tokenization of game assets: The tokenization of game assets, such as the AI fighters, can help create a digital marketplace where players can buy, sell, and trade these assets, potentially creating new revenue streams for the project.

3. Skill-based gameplay: The AI Arena project focuses on skill-based gameplay, which can help attract and retain a dedicated player base who are interested in competitive eSports gaming experiences.

4. Modularity: The contract is divided into several functions, each with a specific purpose. This modularity makes the contract easier to understand, maintain, and test.

Things to Commend:

1. Human x AI collaboration: The AI Arena project introduces the concept of Human x AI collaboration, which is an innovative and unique approach to integrating AI research into a gaming experience.

2. Off-chain game server architecture: By utilizing an off-chain game server architecture, the AI Arena project can potentially help reduce the gas fees associated with on-chain transactions, making the game more accessible and affordable for players.

Weak spots:
Poor Event Logging: The contract does not emit events for state-changing functions. Events are crucial for tracking the contract's activity on the blockchain and can be used for monitoring and debugging. Consider adding events to state-changing functions.
Upgradability: The contract does not appear to be designed with upgradability in mind. For If the contract does needs to be upgraded in the future, consider implementing an upgrade mechanism.

Recommendations:
1. Implement comprehensive security measures: The AI Arena project should prioritize implementing comprehensive security measures to help protect the smart contracts and the overall gaming ecosystem from potential security threats and vulnerabilities.

2. Expand smart contract functionality: The AI Arena project should consider expanding the smart contract functionality to help create a more engaging and immersive gaming experience for players.

4. Optimize off-chain game server architecture: The AI Arena project should focus on optimizing the off-chain game server architecture to help ensure that the game remains accessible and affordable for players, even as the player base grows.

5. Continuously monitor and improve the game: The AI Arena project should prioritize continuously monitoring and improving the game based on player feedback and game analytics data to help ensure that the game remains engaging, fun, and competitive for players.
Overall, the architecture of the codebase appears to be robust and well-thought-out.

Codebase Quality Analysis
Code quality and documentation is very mature. The test suite is pretty comprehensive,providing a solid overview of how AI Arena is structured and how its various aspects function.  Since I found numerous coded POC confirmed issues, the only way I would decide the quality of this codebase is through the issues per contract and design structure; i.e. code readability/maintainability. However, noticed that there is room for additional details;
Input Validation: The codebase lacks comprehensive input validation for many functions, which could potentially lead to security vulnerabilities if an attacker were to provide malicious input values.
Code Readability: The codebase could benefit from improvements in readability, such as better variable naming, more consistent indentation, and clearer function documentation.
Code Duplication: The codebase has some instances of code duplication, which could potentially lead to maintainability issues and increase the likelihood of introducing bugs during code updates and modifications.
Overall, the codebase quality analysis highlights the need for improvements in, code readability, code duplication reduction, code optimization. By addressing these areas for improvement, the codebase quality can be significantly enhanced, leading to a more secure, maintainable, and scalable smart contract ecosystem for the AI Arena project. My overall opinion on the codebase quality analysis based on both issues per contract and code readability/maintainability.

Centralization risks

There is pose for centralization risk and its associated with having a single externally owned account (EOA) as the sole owner of contracts is significant. A single private key, which is controlled by one person, is a single point of failure. If this key is compromised, the entire contract system could be at risk. Additionally, if the owner becomes incapacitated or chooses to act maliciously, the consequences can be dire, including a potential "rug-pull" scenario where the owner could drain all assets from the contract. There are even serveral instances as seen from the bot race
```File: src/FighterFarm.sol

120:     function transferOwnership(address newOwnerAddress) external {

129:     function incrementGeneration(uint8 fighterType) external returns (uint8) {

139:     function addStaker(address newStaker) external {

147:     function instantiateAIArenaHelperContract(address aiArenaHelperAddress) external {

155:     function instantiateMintpassContract(address mintpassAddress) external {

163:     function instantiateNeuronContract(address neuronAddress) external {

171:     function setMergingPoolAddress(address mergingPoolAddress) external {

180:     function setTokenURI(uint256 tokenId, string calldata newTokenURI) external {

268:     function updateFighterStaking(uint256 tokenId, bool stakingStatus) external {

313      function mintFromMergingPool(
314          address to, 
315          string calldata modelHash, 
316          string calldata modelType, 
317          uint256[2] calldata customAttributes
318      ) 
319          public 
320:     {

370:     function reRoll(uint8 tokenId, uint8 fighterType) public {
```
To mitigate these risks, it is recommended to employ a multi-signature setup or a role-based authorization model. Multi-signature requires multiple parties to authorize a transaction, which can prevent a single compromised key from being a significant risk. A role-based model allows for the delegation of authority, enabling different roles to perform different actions, which can be revoked if necessary. This can provide a more distributed control mechanism and reduce the risk of centralization.

Mechanism and Systemic risks Review
Systemic risk in the context of the AI Arena games contract could arise from several factors;
Third-Party Dependency Risk: Contracts rely on external data sources, such as @openzeppelin/contracts, and there is a risk that if any issues are found with these dependencies in your contracts, the Livepeer protocol could also be affected.
Timestamp Dependency Attacks: The lack of comprehensive protection against timestamp dependency attacks in the codebase poses a significant systemic risk. This vulnerability could potentially allow attackers to manipulate the smart contracts and cause financial losses or other negative impacts on the AI Arena project and its community.
Recommendation: Implement robust protection mechanisms against timestamp dependency attacks, such as using `block.number` or `block.timestamp` values instead of relying on the current block timestamp value, to help prevent potential security threats and vulnerabilities.
Interconnectedness: The games contract might be interconnected with other smart contracts or systems that are part of the same ecosystem. For example, if the games contract interacts with a token contract to mint tokens, and if the token contract also interacts with other contracts, the failure of any of these contracts could potentially lead to a cascading failure affecting the entire ecosystem.
Common Exposures: If the games contract and other contracts in the ecosystem use similar risk management practices or models, this could lead to a common exposure. For instance, if they all rely on a similar AI model for some aspect of their functionality, and if this model turns out to be flawed, it could potentially affect all contracts that rely on it.
Procyclicality: The risk of systemic risk can increase during periods of economic stress. For example, if the games contract allows users to stake tokens, and if many users decide to withdraw their stakes during a period of economic stress, this could potentially lead to a liquidity crisis in the token contract that the games contract interacts with.
External Influence: External factors, such as regulatory changes or changes in the market, could potentially affect the operation of the games contract and the entire ecosystem. For example, if new regulations are introduced that affect the operation of the token contract, this could potentially affect the games contract as well.
Smart Contract Vulnerability Risk: Smart contracts can contain vulnerabilities that can be exploited by attackers. If a smart contract has critical security flaws, such as access errors or logic problems, this could lead to asset loss or system manipulation. We strongly recommend that, once the protocol is audited, necessary actions be taken to mitigate any issues identified by C4 Wardens.
To mitigate these risk, the contract could implement measures such as redundancy and fail-safe mechanisms to ensure that the contract can continue to function even if other contracts in the ecosystem fail. It could also implement checks and balances to prevent common exposures, such as by using different AI models for different aspects of its functionality.

Conclusions 

After thoroughly examining the code and considering the architectural choices made in the AI Arena project, it can be concluded that the project demonstrates a commendable effort to balance simplicity and complexity management. The project's smart contracts showcase a strategic approach to simplifying inherently complex systems.
The audit of the contracts within scope has provided a valuable overview of the methodology employed during the development process. The insights gained from this audit can serve as a beneficial resource for the project team and any party interested in analysing this codebase.
By addressing the areas for improvement identified during the audit, the codebase quality can be significantly enhanced, leading to a more secure, maintainable, and scalable smart contract ecosystem for the AI Arena project. Continuously monitoring and improving the game based on player feedback and game analytics data will help ensure that the game remains engaging, fun, and competitive for players.

In summary, the AI Arena project has successfully struck a harmonious balance between the imperative for simplicity and the challenge of managing complexity. The insights gained from the audit of the contracts within scope can serve as a beneficial resource for the project team and any party interested in analysing this codebase. By addressing the areas for improvement identified during the audit, the codebase quality can be significantly enhanced, leading to a more secure, maintainable, and scalable smart contract ecosystem for the AI Arena project.

Time Spent

A total of 6hours daily  for 6days was dedicated to completing this audit





### Time spent:
17 hours