## Overview

AI Arena establishes an intricate gaming economy around tradable NFT fighters powered by machine learning models. Users battle their AIs in a decentralized PvP platform to earn token rewards.

**Architecture**

The core gameplay logic resides off-chain in the game servers, which act as oracles submitting match results on-chain. Smart contracts enforce ownership, ratings, and incentive models.

- [`FighterFarm`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol) - Minting and metadata for NFT fighters 
- [`GameItems`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol) - Purchase in-game assets
- [`RankedBattles`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol) - Staking, match adjudication, rewards distribution
- [`Neuron`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol) - ERC20 utility token for incentives

```
+----------------------------+\
|              UI             |  
+----------------------------+/
|       Front-End App         |
+----------------------------+
|          Oracle            |
|      (Game Server)         |
+----------------------------+
|                            |
|      Smart Contracts       |   
|                            |
+----------------------------+
|          EVM               |
+----------------------------+
\           /
 |  Blockchain  |
/           \
+----------------+
```

The game servers act as the oracle managing fight outcomes and rankings.

**Design Patterns**

- **Access Control** - Permissioned roles like MINTER manage capabilities
- **Checks-Effects-Interactions** - State changes before transfers
- **Oracle** - Game servers provide trusted match data  
- **Reentrancy Guard** - Reentry protection on external calls

**Security Analysis**

- **Custom Errors** - Precise error signaling for front-ends
- **Input Validation** - Enforces match rules and eligibility checks
- **Rate Limiting** - Voltage costs constrain match pace 
- **Events & Modifiers** - Effective state monitoring

But risks remain around unchecked oracle inputs, overpowered admin roles, and economic stability.

**Risk Vectors**

- **Oracle Manipulation** - Unvalidated data could grossly distort rankings and rewards
- **Administer Keys** - Compromised owner keys can siphon excessive value via overrides
- **Economic Sustainability** - Lack of sunk cost constraints, pooled collusions amongst players

**Testing & Audits**

- High unit test coverage targeting individual functions
- Integration testing against live enviornments needed
- No evidence of formal audits or verification yet

**Centralization Risks**

The `RankedBattles` contract shows total trust in the opaque `gameServer` oracle:

No verification checks against the inputs opens manipulation vectors.

**Incentives Review** 

The `Staking` contract enforces a square root curve when rewarding stakes: 

```solidity
points = sqrt(stakeAmount + riskAmount) * eloMultiplier
```

Curbing returns to scale helps sustainability.

- Admins could selectively pick acquaintances or "favored" addresses as winners rather than true randomness 

- The randomness process occurring off-chain has potential for bias influence if improper tools or weak protocols used

- If the selection admin account was compromised, attacker could exploit

Mitigation Strategies:  

- Have an on-chain DAO of trusted community members confirm admin picks before recording 

- Utilize proper tools like Chainlink VRF for performing robust verifiable randomness off-chain

- Separate admin roles - one to run selection, one to confirm and record winners 

- Audit selection software and admin processes regularly for fairness

- Provide transparency to community by publishing details of off-chain selection protocols

- Allow third-party Solidity contract to view winners then encrypt for delayed reveal to mitigate temporary compromise recovery time

By decentralizing control, leveraging trusted blockchain-native randomness, enacting process controls, and pursuing regular auditing - risks around manipulation and bias can be greatly reduced in order to uphold selection integrity. Community involvement also brings additional guardrails.

---

For admins to manipulate the raffle selection process in the MergingPool contract.

The key vulnerability stems from the `pickWinner` function, which relies on admins to provide the array of tokenIds for each period's winners:

```solidity
function pickWinner(uint256[] calldata winners) external {
  require(isAdmin[msg.sender]);

  // Save winner addresses and reset points 
}
```

By placing full trust in admins to supply the winner data, this creates the opportunity for malicious or biased selection:

**Attack Scenario**

1. A compromised admin could call `pickWinner` and cherry-pick acquaintances to "win" the raffle rather than true random addresses

2. Over time, this allows them to disproportionately distribute special NFT rewards to friends rather than the intended random distribution

3. If the attack goes undetected, they could slowly syphon rare NFTs by social engineering

**Root Cause**

The root cause is that the contract code does not actually enforce or validate randomness. It fully trusts the admin's supplied data.

**Impact**

- Unfair distribution skewing rewards over time
- Breaks the intended randomness of the merging pool system
- Devalues rewards for other players who rely on the integrity  

**Mitigations**

- Use a DAO of multiple admins to provide checks and balances 

- Leverage ChainLink VRF to provably inject verified randomness 

- Publish details of off-chain selection for transparency

By addressing the root over-reliance on a single admin, adding transparency, and integrating blockchain-enabled randomness, the risks can be reduced considerably while retaining admin functionality.

While the on-chain `MergingPool` contract records winners, it currently depends on some off-chain process to actually generate that random data:

```
// OFF-CHAIN randomness generation  

function pickWinner(uint256[] calldata winners) external {

  // On-chain recording
}
```

If improper randomness tools or weak protocols are used off-chain, there is risk of predictable or biased results:

**Attack Scenario** 
1. The off-chain selection relies on a simple PRNG with a predictable seed  
2. An attacker can guess or reverse engineer the seed and pre-determine outcomes
3. They use this to prepare game accounts and fighters most likely to "win" the rewards
4. Over time they exploit foreknowledge to siphon rare NFT prizes  

**Root Cause**

The root cause is reliance on unproven or centralized off-chain protocols lacking cryptography. Without rigour here, the on-chain contract is powerless to enforce proper randomness.

**Impact**

- Prize distribution becomes more predictable/exploitable 
- Discourages normal players from participating
- Breaks the key value of randomness and fairness

**Mitigations** 

- Leverage Chainlink VRF for on-chain verifiable randomness
- Conduct third party audits of off-chain systems
- Open source the selection algorithms for community review

This analysis underscores the severe vulnerabilities introduced through opaque off-chain components interacting with on-chain logic. By transitioning to trustless blockchain-based randomness and enabling transparency, we gain significant assurances.


Governance rights in the AI Arena smart contracts are handled in a centralized fashion through the use of admin roles and owner addresses. Here are some key details:

1. Most contracts have an `_ownerAddress` set at deployment via the constructor. 

2. Owner addresses can unilaterally call restricted functions like transferring ownership or adjusting parameters.

3. There is also an `isAdmin` mapping that grants privileges to additional admin addresses.

4. Only owners can add/remove admin role assignments using functions like `addAdmin` or `adjustAdminAccess`.

5. Typical admin privileges include pausing/unpausing functionality, changing key parameters, or distributing tokens.

6. Some execution like winner selection in MergingPool does rely on admins being "honest" actors.

So in summary:
- Governance follows a centralized admin model rather than decentralized community vote-based model.

- Contract ownership and parameterization control rests firmly with the ArenaX team and appointed admins. 

- There are some trusts placed in proper admin execution for key processes.

The contracts do not impose checks and balances or separation of duties on admins. So governance aligns more with an authoritative hierarchy rather than emergent consensus - retaining control for the originating business entity.

**Opportunities**

- **Decreased Oracle Centralization** - Chainlink and other decentralized alternatives
- **Admin Controls** - Multi-sig, time delays, bonding, emergency pausing 
- **Enhanced Validation** - Statistical checks across match results and rankings
- **Formal Verification** - Mathematical proofs of key economic constraints
- **Consensus Mechanisms** - Governance through Aragon, Compound, etc  

AI Arena establishes an exciting foundation spanning machine learning, incentives design, and NFT collateralization. But for long-term sustainability, reducing points of centralization and mathematically proving hyper-incentive alignment will be key. The project shows promise in exploring futuristic intersections of technology, economics, and governance.



### Time spent:
30 hours