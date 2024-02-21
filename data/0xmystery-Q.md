## [L-01] Incomplete array length check
`FighterFarm.redeemMintPass()`'s design requires equal lengths for arrays such as `mintpassIdsToBurn`, `mintPassDnas`, `fighterTypes`, `modelHashes`, and `modelTypes` to ensure each element corresponds across these arrays during the fighter creation process. However, the initial implementation overlooked the `iconsTypes` array, which also plays a significant role in this process. Incorporating a length check for the `iconsTypes` array alongside the existing ones is essential to guarantee that each fighter's creation is based on consistent and complete data, thereby enhancing the contract's reliability and preventing transaction failures due to data mismatches.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L243-L248

```diff
        require(
            mintpassIdsToBurn.length == mintPassDnas.length && 
            mintPassDnas.length == fighterTypes.length && 
            fighterTypes.length == modelHashes.length &&
+            fighterTypes.length == iconTypes.length &&
            modelHashes.length == modelTypes.length
        );
```
## [L-02] Fighters should not be allowed to initiate a fight with zero voltage left 
`RankedBattle.updateBattleRecord()` allows for battle initiation even if the fighter's owner lacks the necessary voltage, provided the voltage replenishment time has passed as indicated in the following checks:

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L334-L338

```solidity
        require(
            !initiatorBool ||
            _voltageManagerInstance.ownerVoltageReplenishTime(fighterOwner) <= block.timestamp || 
            _voltageManagerInstance.ownerVoltage(fighterOwner) >= VOLTAGE_COST
        );
```

The delay could possibly lead to dodging point deduction to claim more NRN tokens for the current round and be leveraged in the next round. The loser fighter could also facilitate unstaking NRN tokens during this window to have 0 `curStakeAtRisk` transferred to `_stakeAtRiskAddress` later that I have reported separately. 

A proposed adjustment to the logic strictly enforces that a fighter's owner must have enough voltage upfront if they are initiating a battle, thereby ensuring that all participants meet the same prerequisites for engagement and maintaining the integrity of the battle system.

## [L-03] Enhancing flexibility in permission management 
`GameItems.setAllowedBurningAddresses()` should introduce a second boolean argument to dynamically grant or revoke burning permissions for addresses, streamlining the administrative process and reducing potential risks. This modification allows for more granular control over permissions, enabling administrators to respond swiftly to changing situations, such as addressing errors or security concerns. This approach not only simplifies the contract's interface but also enhances its adaptability and security, making it a robust solution for managing game item transactions in the AI Arena.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L185-L188

```diff
-    function setAllowedBurningAddresses(address newBurningAddress) public {
+    function setAllowedBurningAddresses(address newBurningAddress, bool isAllowed) public {
        require(isAdmin[msg.sender]);
-        allowedBurningAddresses[newBurningAddress] = true;
+        allowedBurningAddresses[newBurningAddress] = isAllowed;
    }
```
## [L-04] Navigating the shift by ensuring fair transaction ordering in a decentralized Arbitrum
As `Arbitrum` considers moving towards a more decentralized sequencer model, the platform faces the challenge of maintaining its current mitigation of frontrunning risks inherent in a "first come, first served" system. The transition could reintroduce vulnerabilities to transaction ordering manipulation, demanding innovative solutions to uphold transaction fairness. Strategies such as commit-reveal schemes, submarine sends, Fair Sequencing Services (FSS), decentralized MEV mitigation techniques, and the incorporation of time-locks and randomness could play pivotal roles. These measures aim to preserve the integrity of transaction sequencing, ensuring that Arbitrum's evolution towards decentralization enhances its ecosystem without compromising the security and fairness that are crucial for user trust and platform reliability.

## [L-05] Accelerated token minting and sustainability concerns in DAO-governed protocols
With the Neuron token contract initially designed for a modest minting schedule of [5,000 tokens](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L157) twice a week, an escalated minting proposal to 5 million tokens bi-weekly by a DAO-governed consensus (presuming the protocol is headed to that direction) raises significant sustainability questions. Such an aggressive inflation rate would deplete the remaining 300 million mintable supply:

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L36-L43

```solidity
    MAX_SUPPLY -= (INITIAL_TREASURY_MINT + INITIAL_CONTRIBUTOR_MINT); 
```
within approximately 30 weeks or about 6.9 months, starkly contrasting the centuries-spanning timeline under the original minting pace. This rapid approach to reward distribution underscores the critical need for careful consideration of tokenomics and long-term viability in DAO governance decisions, especially when balancing incentive mechanisms against the risk of premature token supply exhaustion.

## [L-06] Enhancing NFT market readiness through immediate metadata initialization
To optimize user experience and ensure NFT marketability from the moment of creation, it's crucial to assign a default token URI to newly minted or rerolled NFTs in FighterFarm.sol just as it has been implemented in [AAMintPass.createMintPass](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AAMintPass.sol#L164-L169). An uninitialized or empty token URI as evidenced from fighter creation and re-roll:

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L529-L530

```solidity
        _safeMint(to, newId);
        FighterOps.fighterCreatedEmitter(newId, weight, element, generation[fighterType]);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L389

```solidity
            _tokenURIs[tokenId] = "";
```
can lead to visibility issues on marketplaces and confusion among users, potentially hindering the immediate trading and appreciation of the NFTs. 

By implementing a default URI pointing to placeholder or generic metadata, and allowing for subsequent updates by [authorized addresses](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L180-L183), projects can maintain flexibility while ensuring that each NFT is market-ready, compliant with standards, and presents seamlessly across platforms and wallets right from its inception. This approach not only enhances user engagement but also upholds the intrinsic value and accessibility of NFTs in dynamic digital marketplaces.

## [L-07] Multi-address exploitation in NFT limitations
The restriction of `MAX_FIGHTERS_ALLOWED = 10` per address as imposed in [FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L33) aims to prevent hoarding and ensure fair distribution among participants. However, this measure can be circumvented by users holding NFTs across multiple addresses, undermining the intended balance and accessibility. Addressing this challenge requires a nuanced strategy that may include social verification, economic disincentives, and participation limits, while carefully balancing user privacy and the decentralized ethos of blockchain. Crafting an effective solution demands a thoughtful approach that deters exploitation without compromising the core values of autonomy and inclusivity inherent to the decentralized community.

Nonetheless, it's understood that the current measure is making way to owners who might need to house more than 10 fighters as they continue emerging as winners from the [merging pool](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L154-L159).

## [L-08] A robust input array length check
Further to the Bot's [L-06], here's a useful code refactoring on `MergingPool.claimRewards()` by aligning the lengths of input arrays (`modelURIs`, `modelTypes`, and `customAttributes`) with the actual number of unclaimed rewards for the claimant.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L134-L185

```diff
    function claimRewards(
        string[] calldata modelURIs, 
        string[] calldata modelTypes,
        uint256[2][] calldata customAttributes
    ) 
        external 
    {
+        uint256 numUnclaimedRewards = getUnclaimedRewards(msg.sender); // Calculate the number of unclaimed rewards
+        equire(modelURIs.length == numUnclaimedRewards && modelTypes.length == numUnclaimedRewards && customAttributes.length == numUnclaimedRewards, "Mismatch in claim data");
        ...
``` 
## [L-09] Front-running vulnerability in ERC20's `approve()`
The `approve` function in Openzeppelin's ERC20.sol (e.g. as inherited by [Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L4)) is inherently susceptible to front-running attacks, where a malicious actor can exploit the window between the submission and confirmation of a transaction to change allowances. This vulnerability emerges prominently when altering an existing non-zero allowance, potentially allowing an attacker to spend more than the intended amount by front-running the transaction with a higher gas fee. To mitigate this risk, the "approve-reset" pattern such as using the `increaseAllowance` and `decreaseAllowance` functions provided by some ERC20 implementations is recommended, involving resetting the allowance to zero before setting it to a new value, despite the inconvenience of requiring two separate transactions. This highlights the importance of understanding and addressing potential security pitfalls in smart contract design, particularly in the widely-used ERC20 token standard.

## [NC-01] Avoid code logic redundancy
In the AI Arena Helper smart contract, the constructor is designed to set up initial attribute probabilities for generation 0 fighters, a critical step for defining the physical characteristics of AI fighters. However, the current implementation exhibits redundancy by first invoking the `addAttributeProbabilities` function and then directly assigning probabilities to the `attributeProbabilities` mapping for each attribute within a loop. This clutters of contract code could be simplified by solely relying on the `addAttributeProbabilities` function for initializing probabilities, improving code readability and thereby streamlining the initialization process in the contract's setup.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L41-L52

```diff
    constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
-            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 
```
## [NC-02] Redundancy checks as a double-edged sword
In Solidity smart contracts, particularly those inheriting from OpenZeppelin's ERC20 implementation, redundancy in checks, such as the allowance verification in a custom `Neuron.claim()` and again in the `transferFrom` method, may appear superfluous but serves critical roles in clarity, security, and early failure handling. While such redundancy can slightly elevate gas costs for successful transactions, it ensures that contracts fail fast on allowance issues, potentially saving gas and providing clearer error messages. This dual-layered approach to checks, especially in token-related functions, underscores the balance developers must strike between gas efficiency and contract robustness, highlighting the nuanced considerations in smart contract design and optimization.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L138-L145

```solidity
    function claim(uint256 amount) external {
        require(
            allowance(treasuryAddress, msg.sender) >= amount, 
            "ERC20: claim amount exceeds allowance"
        );
        transferFrom(treasuryAddress, msg.sender, amount);
        emit TokensClaimed(msg.sender, amount);
    }
```
## [NC-03] Activate the Optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.0",
settings: {
optimizer: {
  enabled: true,
  runs: 1000,
},
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.
