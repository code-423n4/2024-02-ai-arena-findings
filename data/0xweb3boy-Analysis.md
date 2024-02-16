
# AI Arena Analysis

## Overview
In the AI Arena - Gaming Competition, participants engage in a dynamic gaming environment where they have the opportunity to purchase, train, and engage in battles with AI-enabled Non-Fungible Tokens (NFTs) within a Player versus Player (PvP) fighting game.

This gaming platform offers a multifaceted experience, where users can acquire NFTs equipped with artificial intelligence capabilities. These NFTs serve as virtual entities that can be nurtured and customized through various training mechanisms, enabling gamers to enhance their abilities and strategize for competitive battles.

Once equipped with their trained NFTs, participants enter the PvP arena, where they engage in thrilling battles against opponents. These battles are characterized by strategic maneuvers, real-time decision-making, and the utilization of unique AI-driven abilities embedded within each NFT.

The AI Arena fosters a competitive gaming atmosphere, where players can showcase their skills, develop effective battle strategies, and strive for victory in exhilarating PvP encounters. Through the integration of AI technology and NFT mechanics, the AI Arena offers a dynamic and immersive gaming experience for enthusiasts seeking engaging and strategic gameplay.

``Smart Contracts``: Utilizes contracts like RankedBattle.sol  which enables users to stake NRN tokens on fighters, monitors battle records, computes and disburses rewards based on battle results and staked quantities, and permits the claiming of accumulated rewards.

``Neuron Token``:Within the platform, the Neuron token serves multiple purposes, including but not limited to staking, governance participation, and reward distribution..

``Risk Management``: With the help of StakeAtRisk this smart contract enables the RankedBattle functionality to oversee the staking process of NRN tokens during battle engagements.


## Project’s Risk Model

## With the help of `updateAtRiskRecords()` function it updates stake at risk records for a fighter avoiding the risks of unintentional loss of points or NRNs.

```solidity

function updateAtRiskRecords(
        uint256 nrnToPlaceAtRisk, 
        uint256 fighterId, 
        address fighterOwner
    ) 
        external 
    {
        require(msg.sender == _rankedBattleAddress, "Call must be from RankedBattle contract");
        stakeAtRisk[roundId][fighterId] += nrnToPlaceAtRisk;
        totalStakeAtRisk[roundId] += nrnToPlaceAtRisk;
        amountLost[fighterOwner] += nrnToPlaceAtRisk;
        emit IncreasedStakeAtRisk(fighterId, nrnToPlaceAtRisk);
    }   

```




### Minting and Staking Dynamics

The mechanism for staking gUSDC to mint GUILD, and the reverse process, might be influenced by the changing values of these tokens. A decline in GUILD's value, for instance, could reduce the incentive for users to stake gUSDC.



## Admin abuse risks

#### Roles

```solidity

        // Initial roles setup: direct hierarchy, everything under governor
        _setRoleAdmin(CoreRoles.GOVERNOR, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.GUARDIAN, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.CREDIT_MINTER, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.RATE_LIMITED_CREDIT_MINTER, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.GUILD_MINTER, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.RATE_LIMITED_GUILD_MINTER, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.GAUGE_ADD, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.GAUGE_REMOVE, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.GAUGE_PARAMETERS, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.GAUGE_PNL_NOTIFIER, CoreRoles.GOVERNOR);
        _setRoleAdmin(
            CoreRoles.GUILD_GOVERNANCE_PARAMETERS,
            CoreRoles.GOVERNOR
        );
        _setRoleAdmin(
            CoreRoles.GUILD_SURPLUS_BUFFER_WITHDRAW,
            CoreRoles.GOVERNOR
        );
        _setRoleAdmin(
            CoreRoles.CREDIT_GOVERNANCE_PARAMETERS,
            CoreRoles.GOVERNOR
        );
        _setRoleAdmin(CoreRoles.CREDIT_REBASE_PARAMETERS, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.TIMELOCK_PROPOSER, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.TIMELOCK_EXECUTOR, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.TIMELOCK_CANCELLER, CoreRoles.GOVERNOR);

```
Sets up a role-based access control system in a Ethereum Credit Guild, with a strong centralization of power under the GOVERNOR role

#### Risk of Admin Abuse

- The ``GOVERNOR`` has administrative power over all roles, potentially leading to a single point of control and decision-making. This centralization can be efficient for management but poses risks of abuse or mismanagement.

- Centralization of control makes the system vulnerable to the decisions or actions of the GOVERNOR. If the GOVERNOR acts maliciously or is compromised, the entire protocol could be at risk.

- With the GOVERNOR controlling all key roles, including those related to minting tokens (CREDIT_MINTER, GUILD_MINTER), managing gauges (GAUGE_ADD, GAUGE_REMOVE, GAUGE_PARAMETERS), and executing time-locked transactions (TIMELOCK_PROPOSER, TIMELOCK_EXECUTOR, TIMELOCK_CANCELLER), there's a risk that this power could be used in ways that are not in the best interest of the protocol or its users.

#### Future Scalability and Adaptability

``Challenges in Protocol Evolution``: As the protocol grows and evolves, the centralized GOVERNOR structure might become a bottleneck, hindering the adoption of changes and updates that require a broader consensus.

``Potential for Governance Gridlock`` : Over time, the concentration of power might lead to governance gridlock if the GOVERNOR becomes resistant to changes or if there’s a conflict of interest within the community.


##

## Architecture assessment of business logic

I analyzed the architecture and business logic of abstract contract ``ERC20RebaseDistributor`` and two important contracts: ``AuctionHouse`` and ``LendingTerm``. This analysis focuses on their structures and operational mechanics.

### ERC20RebaseDistributor Contract

#### Architecture

![ERC20RebaseDistributor](https://gist.github.com/assets/58845085/906c9cea-c326-4dcf-9445-42285e7a30be)

The ERC20RebaseDistributor contract, designed for a rebase mechanism in a DeFi protocol, exhibits several key implementations,

``Rebase Functionality``: It allows token supply adjustments (rebasing), proportionally affecting the balance of all token holders who opt into rebasing.

``Opt-in Mechanism``: Users can choose to participate in rebasing through enterRebase() and opt out using exitRebase(), providing flexibility in how they wish to interact with the token's economics.

``Token Burn for Distribution``: The contract enables users to burn their tokens to distribute value to all rebasing token holders, enhancing the token's utility as a distribution mechanism.

``Share-Based Accounting``: Internally, the contract converts user balances to shares when they opt into rebasing. This approach simplifies the calculation of individual holdings post-rebase.

``Dynamic Share Price Calculation``: The share price is recalculated upon each distribution, ensuring that the value of rebasing shares accurately reflects the new total supply.

``Interpolation for Distribution``: The contract uses a linear interpolation over a set period (30 days) to gradually distribute the value of burned tokens, preventing sudden supply shocks.

``Safeguards Against Manipulation``: The contract highlights the need for a meaningful initial balance in rebasing to prevent share price manipulation, echoing concerns from historical DeFi exploits.

### AuctionHouse Contract

#### Architecture

![ActionHouse](https://gist.github.com/assets/58845085/e7dff48e-1ad6-47c0-bb55-6f0bf510754f)

#### AuctionHouse Intraction Flow

![AuctionHouse](https://gist.github.com/assets/58845085/8f61b2ff-6c35-4e47-bac8-48126883fde7)

![AuctionHouseIntraction](https://gist.github.com/assets/58845085/adde9417-a15f-499c-a345-bc58c2685b12)

The AuctionHouse contract in a DeFi protocol typically manages the auctioning of assets, often used for liquidating collateral from defaulted loans or for other decentralized governance purposes. It allows users to place bids on assets, handles the logic for determining winning bids, and ensures the transfer of both assets and payment according to the auction rules. The contract is designed to be transparent, fair, and efficient, providing a decentralized mechanism for asset liquidation or sales within the ecosystem

### LendingTerm Contract

#### Architecture

![LendingTerm1](https://gist.github.com/assets/58845085/aa5a454b-dba4-4977-a632-095fa65e1b54)

#### LendingTerm Intraction Flow

![LendingTerm](https://gist.github.com/assets/58845085/b90b6079-6f1b-4d0e-88ae-db3aee2c5665)

The LendingTerm contract in a DeFi protocol is designed to define and manage the terms of lending operations.

``Setting Loan Parameters``: It specifies the conditions for loans, such as interest rates, collateral requirements, maximum debt per collateral, and repayment schedules.

``Loan Origination``: Facilitates the creation of new loans under the specified terms, handling collateralization and issuance of debt.

``Repayment and Liquidation``: Manages loan repayments and enforces liquidation in case of defaults, ensuring compliance with the set terms.

``Governance Integration``: Often integrated with governance mechanisms for modifying terms or adding new ones, reflecting the decentralized decision-making process.


## Testing Analysis

| File                                              | % Lines            | % Statements       | % Branches       | % Funcs           |
|---------------------------------------------------|--------------------|--------------------|------------------|-------------------|
| src/core/Core.sol                                 | 100.00% (1/1)      | 100.00% (1/1)      | 100.00% (0/0)    | 100.00% (1/1)     |
| src/core/CoreRef.sol                              | 100.00% (15/15)    | 100.00% (20/20)    | 100.00% (2/2)    | 100.00% (6/6)     |
| src/governance/GuildGovernor.sol                  | 92.86% (13/14)     | 92.86% (13/14)     | 100.00% (0/0)    | 92.31% (12/13)    |
| src/governance/GuildTimelockController.sol        | 100.00% (1/1)      | 100.00% (1/1)      | 100.00% (0/0)    | 25.00% (1/4)      |
| src/governance/GuildVetoGovernor.sol              | 96.49% (55/57)     | 96.72% (59/61)     | 90.00% (18/20)   | 94.44% (17/18)    |
| src/governance/LendingTermOffboarding.sol         | 100.00% (33/33)    | 100.00% (34/34)    | 92.86% (26/28)   | 100.00% (5/5)     |
| src/governance/LendingTermOnboarding.sol          | 100.00% (36/36)    | 100.00% (40/40)    | 100.00% (22/22)  | 100.00% (5/5)     |
| src/governance/ProfitManager.sol                  | 100.00% (117/117)  | 100.00% (136/136)  | 86.96% (40/46)   | 100.00% (15/15)   |
| src/loan/AuctionHouse.sol                         | 100.00% (38/38)    | 100.00% (44/44)    | 95.00% (19/20)   | 100.00% (5/5)     |
| src/loan/LendingTerm.sol                          | 100.00% (210/210)  | 98.82% (251/254)   | 87.30% (110/126) | 100.00% (24/24)   |
| src/loan/SimplePSM.sol                            | 100.00% (25/25)    | 100.00% (27/27)    | 100.00% (4/4)    | 100.00% (7/7)     |
| src/loan/SurplusGuildMinter.sol                   | 100.00% (83/83)    | 98.99% (98/99)     | 85.00% (34/40)   | 100.00% (7/7)     |
| src/rate-limits/RateLimitedMinter.sol             | 100.00% (3/3)      | 100.00% (3/3)      | 100.00% (0/0)    | 100.00% (2/2)     |
| src/tokens/CreditToken.sol                        | 100.00% (16/16)    | 100.00% (16/16)    | 100.00% (4/4)    | 100.00% (11/11)   |
| src/tokens/ERC20Gauges.sol                        | 100.00% (123/123)  | 100.00% (133/133)  | 92.00% (46/50)   | 100.00% (28/28)   |
| src/tokens/ERC20MultiVotes.sol                    | 100.00% (91/91)    | 100.00% (102/102)  | 73.53% (25/34)   | 100.00% (27/27)   |
| src/tokens/ERC20RebaseDistributor.sol             | 99.49% (195/196)   | 99.62% (263/264)   | 98.39% (61/62)   | 100.00% (23/23)   |
| src/tokens/GuildToken.sol                         | 100.00% (51/51)    | 100.00% (53/53)    | 94.44% (17/18)   | 100.00% (17/17)   |
| src/utils/RateLimitedV2.sol                       | 100.00% (37/37)    | 100.00% (46/46)    | 100.00% (10/10)  | 100.00% (8/8)     |
| test/forge-std/lib/ds-test/src/test.sol           | 0.00% (0/219)      | 0.00% (0/222)      | 0.00% (0/110)    | 0.00% (0/56)      |
| test/forge-std/src/DSTest.sol                     | 11.87% (26/219)    | 13.06% (29/222)    | 11.82% (13/110)  | 12.50% (7/56)     |
| test/forge-std/src/StdAssertions.sol              | 98.90% (90/91)     | 95.15% (98/103)    | 90.00% (36/40)   | 95.45% (21/22)    |
| test/forge-std/src/StdCheats.sol                  | 0.00% (0/188)      | 0.00% (0/228)      | 0.00% (0/52)     | 0.00% (0/38)      |
| test/forge-std/src/StdJson.sol                    | 0.00% (0/29)       | 0.00% (0/29)       | 100.00% (0/0)    | 0.00% (0/29)      |
| test/forge-std/src/StdMath.sol                    | 0.00% (0/12)       | 0.00% (0/15)       | 0.00% (0/4)      | 0.00% (0/5)       |
| test/forge-std/src/StdStorage.sol                 | 0.00% (0/127)      | 0.00% (0/153)      | 0.00% (0/30)     | 0.00% (0/37)      |
| test/forge-std/src/StdUtils.sol                   | 0.00% (0/31)       | 0.00% (0/40)       | 0.00% (0/26)     | 0.00% (0/5)       |
| test/forge-std/src/console.sol                    | 0.00% (0/381)      | 0.00% (0/381)      | 100.00% (0/0)    | 0.00% (0/380)     |
| test/forge-std/src/console2.sol                   | 0.00% (0/381)      | 0.00% (0/381)      | 100.00% (0/0)    | 0.00% (0/380)     |
| test/forge-std/test/StdAssertions.t.sol           | 100.00% (21/21)    | 100.00% (21/21)    | 100.00% (0/0)    | 100.00% (21/21)   |
| test/forge-std/test/StdCheats.t.sol               | 100.00% (5/5)      | 100.00% (5/5)      | 50.00% (5/10)    | 100.00% (3/3)     |
| test/forge-std/test/StdError.t.sol                | 92.31% (12/13)     | 93.33% (14/15)     | 50.00% (1/2)     | 100.00% (10/10)   |
| test/forge-std/test/StdStorage.t.sol              | 60.00% (3/5)       | 66.67% (4/6)       | 100.00% (0/0)    | 50.00% (2/4)      |
| test/integration/PostProposalCheck.sol            | 0.00% (0/6)        | 0.00% (0/6)        | 100.00% (0/0)    | 0.00% (0/1)       |
| test/integration/PostProposalCheckFixture.sol     | 0.00% (0/31)       | 0.00% (0/32)       | 100.00% (0/0)    | 0.00% (0/1)       |
| test/mock/MockERC20.sol                           | 66.67% (6/9)       | 66.67% (6/9)       | 100.00% (0/0)    | 57.14% (4/7)      |
| test/mock/MockERC20Gauges.sol                     | 87.50% (7/8)       | 87.50% (7/8)       | 100.00% (0/0)    | 87.50% (7/8)      |
| test/mock/MockERC20MultiVotes.sol                 | 85.71% (6/7)       | 85.71% (6/7)       | 100.00% (0/0)    | 85.71% (6/7)      |
| test/mock/MockERC20RebaseDistributor.sol          | 85.71% (6/7)       | 85.71% (6/7)       | 100.00% (0/0)    | 85.71% (6/7)      |
| test/mock/MockRateLimitedV2.sol                   | 100.00% (2/2)      | 100.00% (2/2)      | 100.00% (0/0)    | 100.00% (2/2)     |
| test/proposals/AddressLib.sol                     | 0.00% (0/46)       | 0.00% (0/63)       | 0.00% (0/10)     | 0.00% (0/4)       |
| test/proposals/gips/GIP_0.sol                     | 0.00% (0/209)      | 0.00% (0/258)      | 100.00% (0/0)    | 0.00% (0/5)       |
| test/proposals/gips/GIP_X.sol                     | 100.00% (0/0)      | 100.00% (0/0)      | 100.00% (0/0)    | 0.00% (0/5)       |
| test/proposals/proposalTypes/MultisigProposal.sol | 0.00% (0/7)        | 0.00% (0/10)       | 0.00% (0/2)      | 0.00% (0/3)       |
| test/proposals/proposalTypes/Proposal.sol         | 0.00% (0/1)        | 0.00% (0/1)        | 100.00% (0/0)    | 0.00% (0/1)       |
| test/proposals/proposalTypes/TimelockProposal.sol | 0.00% (0/38)       | 0.00% (0/43)       | 0.00% (0/18)     | 0.00% (0/3)       |
| Total                                             | 40.96% (1327/3240) | 42.54% (1538/3615) | 54.78% (493/900) | 23.38% (310/1326) |


### 100% Coverage:

Files like Core.sol, CoreRef.sol, LendingTermOnboarding.sol, and several others show 100% coverage across lines, statements, branches, and functions. This indicates that every line of code, every conditional branch, and every function in these files have been executed during testing, which is ideal.

### Less Than 100% Coverage:

Files such as GuildGovernor.sol, GuildVetoGovernor.sol, and LendingTerm.sol have less than 100% coverage in some areas. This could be a cause for concern, especially in a DeFi protocol, where untested code might lead to vulnerabilities or unexpected behaviors.
The test/forge-std/... files have significantly lower coverage. These might be standard test utilities or mock contracts used for testing other components. While lower coverage here might be less critical, it's still worth investigating if it impacts the reliability of the tests.

### Impact of Non-100% Coverage:

``Security Risks``: In DeFi, untested code could lead to security vulnerabilities, potentially leading to loss of funds or other exploits.
``Functional Bugs``: Lack of coverage might mean that certain edge cases or error conditions are not tested, which could result in bugs or unexpected behavior in production.
``Integration Issues``: In a protocol with multiple interacting contracts, untested code in one contract might cause issues in another, especially in complex transactions or state changes.

## Software engineering considerations

### Purpose-Specific Modules
Design contracts like ERC20RebaseDistributor, AuctionHouse, and LendingTerm as modular components. This allows for easier upgrades and maintenance.

### Separation of Concerns
Ensure each contract focuses on a specific functionality, reducing complexity and potential for bugs.

### Unit Testing
Develop comprehensive unit tests for each function in the contracts, covering all possible edge cases.

### Stress Testing and Simulation
Perform stress tests simulating extreme market conditions to evaluate the contracts' resilience.

### Mathematical Proofs
Use formal verification tools to prove the correctness of critical functions, especially those in ERC20RebaseDistributor involving complex mathematical calculations.

### State Invariants
Establish and verify state invariants for contracts to ensure they maintain consistent states throughout transactions.

### Reentrancy Guards
Implement checks against reentrancy attacks, particularly in functions that interact externally in AuctionHouse.

### Optimize for Gas
Refactor contract code to minimize gas consumption, crucial for functions likely to be called frequently.

### Proxy Patterns
If upgradeability is a requirement, consider using proxy patterns, ensuring a clear separation between data and logic.

### Version Control
Maintain strict version control and changelogs for all contracts to track changes and upgrades over time.

### Transparent Governance Processes
Develop and maintain clear, transparent processes for governance actions, documented in code comments and developer documentation.

### Smart Contract Events
Emit detailed events in contracts for better off-chain tracking and to facilitate user interface updates.

### Comprehensive Documentation
Maintain detailed documentation for each contract, describing its purpose, functions, and interaction mechanisms.


## Code Weakspots 

### Complex Rebasing Logic in _balance2shares and _shares2balance Functions

```solidity

function _balance2shares(uint256 balance, uint256 sharePrice) internal pure returns (uint256) {
    return (balance * START_REBASING_SHARE_PRICE) / sharePrice;
}

function _shares2balance(uint256 shares, uint256 sharePrice, uint256 deltaBalance, uint256 minBalance) internal pure returns (uint256) {
    uint256 rebasedBalance = (shares * sharePrice) / START_REBASING_SHARE_PRICE + deltaBalance;
    if (rebasedBalance < minBalance) {
        rebasedBalance = minBalance;
    }
    return rebasedBalance;
}

```
#### Why It's Weak:
These functions handle the conversion between balances and shares, which is a core part of the rebasing logic. The risk here is primarily due to potential rounding errors and integer division limitations in Solidity. Incorrect conversions could lead to balance inaccuracies, especially when dealing with large numbers or very small fractional amounts.

### Complexity in getBidDetail() Function

```solidity

function getBidDetail(bytes32 loanId) public view returns (uint256 collateralReceived, uint256 creditAsked) {
    // ... logic for calculating bid details ...
}

```
#### Why It's Weak:
The getBidDetail function calculates how much collateral is offered and how much debt is asked for, based on the time elapsed in the auction. The logic is complex and time-dependent, making it prone to miscalculations or manipulation, especially if not properly validated against edge cases or unexpected scenarios (like extreme market conditions).

### Handling of Negative Values in notifyPnL

```solidity

if (amount < 0) {

```
#### Why It's Weak:
The handling of losses (negative values) is complex and involves multiple state changes (e.g., adjusting surplus buffers, updating credit multipliers). If not managed accurately, it could lead to financial discrepancies in the system, such as incorrect loss accounting or multiplier adjustments.

### creditMultiplier Risk

```solidity

// update the CREDIT multiplier
                uint256 creditTotalSupply = CreditToken(_credit).totalSupply();
                uint256 newCreditMultiplier = (creditMultiplier *
                    (creditTotalSupply - loss)) / creditTotalSupply;
                creditMultiplier = newCreditMultiplier;
                emit CreditMultiplierUpdate(
                    block.timestamp,
                    newCreditMultiplier
                );

```
### Calculating the New Multiplier
- creditTotalSupply is the total supply of CREDIT tokens in the system.
- loss is the amount of bad debt that needs to be accounted for, beyond the surplus buffer.
- The new creditMultiplier is calculated by reducing the current multiplier proportionally to the ratio of the loss to the total CREDIT supply.

### Effect of the Calculation:
- The formula (creditMultiplier * (creditTotalSupply - loss)) / creditTotalSupply effectively reduces the creditMultiplier.
- If loss is significant compared to creditTotalSupply, this reduction can be substantial.
- For example, if the current multiplier is 1.0 (or 1e18 in Solidity's integer math), and the loss is 10% of the total CREDIT supply, the new multiplier would be 0.9 (or 0.9e18), representing a 10% decrease.

### Implications:
``Decrease in CREDIT Value``: The decrease in the creditMultiplier effectively lowers the value of CREDIT tokens throughout the system. This means that every CREDIT token now represents a smaller share of the system's value.

``Increased Debt for Borrowers``:  This decrease means borrowers now owe more CREDIT tokens to cover the same nominal value of debt.












### Time spent:
30 hours



### Time spent:
30 hours