
# Contract Overview

#### AiArenaHelper:

The AiArenaHelper contract is a helper contract that generates and manages Ai Arena fighter's physical attributes. The contract includes functionality to handle ownership, attribute probabilities, and the creation of physical attributes based on a fighter's DNA. It is used in other contracts to facilitate the generation of Ai fighters.

#### FighterOps:

This library manages the creation and attributes of Fighter NFTs within the game. The library is designed to be used by other contracts to retrieve the relevant data for Ai fighters. The library includes the data structures that define each fighter and includes relevant functions to expose this data publicly.

#### GameItems:

The GameItems smart contract inherits from OpenZeppelin's ERC-1155 standard allowing for the creation of fungible and non-fungible tokens within a single contract. The contract is designed to manage game items for Ai Arena. The GameItems smart contract allows for trusted admins to create unique game items, and allows anyone to mint these items as long as pre-set requirements are fulfilled. When creating an item, the admin can set the transferability, price, daily allowance, and supply of the game items. The supply can be infinite if desired. The contract also defines the logic for transferring and burning game items, while complying with uri requirements set by ERC-1155. Payment for game items is done through the Neuron ERC-20 token.

#### MergingPool:

The MergingPool contract includes functionality for managing points associated with fighter tokens, selecting winners, and claiming rewards. Users earn points by winning rounds through the RankedBattle contract, which calls the addPoints function of the MergingPool contract. This contract also allows admin accounts to select the winners of a round, and allows the winners to claim rewards, which are additional Ai fighters.

#### Neuron:

The Neuron contract is the payment system for Ai Arena, and is an ERC-20 token with a maximum supply of 10e27 and 18 decimals.

#### VoltageManager:

The VoltageManager contract is designed to manage a "voltage" system for AI Arena. The voltage system is effectively a stamina system seen in other online games. Users are given a set amount of voltage to use per day, and actions like battling consume voltage. Voltage can also be replenished through voltage batteries, and are sold through the GameItems contract. The voltage system can be extended to be used in other aspects of the Ai Arena ecosystem.

#### StakeAtRisk:

The StakeAtRisk contract is designed to manage the stakes that are at risk of being lost during a round of competition in Ai Arena. The contract keeps track of the stakes that fighters have at risk in a given round, allows for the reclamation of tokens if a fighter wins, and updates the records of at-risk stakes when a fighter loses. At the end of a round, it sweeps all lost stakes to a treasury address.

#### RankedBattle:

The RankedBattle contract defines the logic for the ranked system of Ai Arena. The contract allows users to stake their Neuron tokens. Users can stake Neuron tokens which is a condition to earn specific rewards when winning a round. Winning a round without staking increases the fighter's points, while winning a round reclaims some of the stake at risk for the user. Losing while having a positive points balance deducts points from the user, while having a zero point balance will move some of the staked Neuron to the StakeAtRisk contract.

#### FighterFarm:

The FighterFarm contract is an ERC-721 token used to represent the fighters themselves. The contract manages the creation, ownership, and trading of Ai Arena fighter NFTs. It allows users to mint new NFTs, claim pre-determined fighters, reroll existing fighters, and burn mint passes in exchange for fighters.


## Centralization Risks:

Ai Arena is a highly centralized project in which the owners have nearly unlimited control over the execution and logic of the game. The owners control the game servers, select the winners and losers, and can replace most of the contracts with a new implementation if desired. Ai Arena is designed as a centralized system that uses blockchain for revenue generation and general distribution. User ownership of the underlying tokens is not absolute.

Due to the centralization of the project, should a trusted role become compromised, the project could suffer significant consequences. Additional information can be found in the section on Access Control - Risk Mitigation.

# System Design

### Modularity
AiArena uses a modular design, where individual aspects of the system are mostly contained in their own separate contracts. There is some overlap in contracts like FighterFarm.sol, but for the most part, it remains modular by design. Some contracts include logic to replace contract instances with new ones at the discretion of the _ownerAddress which allows for effectively upgrading these contracts. A word of caution to remember when switching contract instances is that all smart contracts have their own storage which will not be replicated to a new instance. This can be remediated by setting the variables in the constructor, but this can become prohibitively gas expensive. This could become an issue due to how these contracts handle state variables, which is discussed more in its own section. A more advanced implementation might include beacon proxies, which could be considered if the system grows in size and complexity. While not an ideal method for allowing upgrades, the implementation is reasonable for this project. 

### Testing
AiArena mostly uses unit testing with magic numbers in the testing of their system. This can lead to false confidence in the robustness of the testing. The test names are verbose and explain the purpose of the tests, although as a whole, they mainly check the surface level of the functions themselves. I would encourage extending the test suite by including tests that call the existing unit tests in different orders, as I managed to discover interesting errors without writing any additional code. I would also recommend adding more fuzz and integration tests to better discern issues in the codebase.

### State
One of the most important parts of writing smart contracts, and programming in general is the management of state. Where data is stored, and how it gets updated is the majority of smart contract development. State fragmentation commonly results in bugs if the state is not synced properly between contracts. Ideally, there is a single state that is referenced by everything in the system to prevent such issues, although for some systems, this becomes impractical or infeasible. The current implementation of AiArena has excessive state fragmentation particularly in its access control, which is described in more detail in its own section. Some state variables are set in multiple contracts like roundId which then requires being updated in multiple places through cross-contract calls. As it seems some of these contracts are supposed to be updated in the future through function calls like setStakeAtRiskAddress, these state variables may need to be reinitialized during deployment becoming gas expensive or effectively impossible altogether. This is different compared to the more common proxy pattern, as the storage is kept in the proxy rather than the implementation. Overall, how AiArena handles state has considerable room for improvement.


### Access Control

The project uses an ill-advised access control mechanism, which is a weakness of the project. The root of the issue is a lack of proper state management, which is evident throughout the smart contracts, and discussed more in its own section.

The project uses several access roles with control over different aspects of the system. Systems with the need for multiple levels of access control traditionally use Openzeppelin's AccessControl contract, or one of its derivatives. With the release of Openzeppelin package version 5.0, they released the AccessManager contract, a contract tailored to managing multiple contracts from a single manager contract. Either option is viable for implementation.

The Neuron contract does import Openzeppelin's AccessControl contract, although its implementation is poor. The Neuron contract does not leverage the onlyRole modifier and instead opts to use require statements to restrict access. Additionally, the implementation uses depreciated functions (\_setupRole Neuron:#L95,103,111).

The ecosystem has several roles:

\_ownerAddress:
A psuedo role with super-admin permissions set to the initial deployer of the contract. This user is set as an admin Neuron:#L73 and can set and remove admins Neuron:#L118-121. Hopefully this is transferred to a multi-sig wallet, as its permissions are absolute.

admin:
Not an admin in the traditional sense, as its permissions are somewhat limited. The admin role is not restricted to a single address, and a state with no admins can be achieved.

MINTER_ROLE:
The MINTER_ROLE is in charge of minting Neuron tokens and is expected to be given to the RankedBattle contract as shown in RankedBattle.t.sol.

SPENDER_ROLE:
The SPENDER_ROLE is permissioned to call the \_approve function of any account for any amount of Neuron tokens. This role is expected to be given to the GameItems contract, as shown in GameItems.t.sol.

STAKER_ROLE:
The STAKER_ROLE is permissioned to call the \_approve function of any account for any amount of Neuron tokens. This role is expected to be given to the RankedBattle contract, as shown in RankedBattle.t.sol.

allowedVoltageSpenders:
The allowedVoltageSpenders is a mapping in the VoltageManager contract that allows addresses to spend the voltage of users.

\_gameServerAddress
The \_gameServerAddress is a single address allowed to call the updateBattleRecord function in the RankedBattle contract.

\_delegatedAddress
The \_delegatedAddress is a single address allowed to update the TokenURI in the FighterFarm contract. It is also used in the hash for signature verification in the claimFighters function.

One major issue the current implementation is regarding state and access control. Each deployed contract keeps track of its own admin mapping, \_ownerAddress, and any other included roles. An admin added to the FighterFarm contract is not also added to the Neuron contract, and vice-versa. Keeping track of which addresses have which roles in each contract is difficult, and can easily lead to instances of improper access of crucial functions.

Instead, all contracts should reference the same access control state governed by a single AccessControl or AccessManager contract. In this manner, the code for adjusting access control need not be repeated in each contract. In the current implementation there are seven instances of the following function:

```
    /// @notice Transfers ownership from one address to another.
    /// @dev Only the owner address is authorized to call this function.
    /// @param newOwnerAddress The address of the new owner
    function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
    }

```

Other functions like adjustAdminAccess are repeated several times as well. These can all be reduced to a single instance in the AccessControl contract. Regardless if the intended functionality of the contract is that admins of one contract are not admins of another contract, this same effect is easily achieved using the role-based access control offered by Openzeppelin's implementations.

The current implementation of role based access is missing a crucial component. Role-based access achieves two major purposes: separation of responsibilities, and risk mitigation.

#### Separation of Responsibilities

Each role should control a specific action or group of similar actions, ideally without overlap. One example of this is the MINTER_ROLE, which is required for the minting of tokens and nothing else. If an entity requires the ability to mint tokens, it must be given the MINTER_ROLE through a higher-level role like an admin, or in this case \_ownerAddress. In the current implementation, there is significant overlap between the \_ownerAddress and the admin role, and permissions for each seem somewhat arbitrary. There is also significant overlap in the STAKER_ROLE and the SPENDER_ROLE. Both are able to approve an arbitrary amount of Neuron tokens for any address.

#### Risk Mitigation

Access control allows for roles to be distinguished based on their potential impact on the protocol. High-impact roles like admins can be separated from roles like allowedVoltageSpenders, where if the former is compromised, the protocol may be irrecoverable, whereas if the latter is compromised the lasting damage to the protocol is minor or nonexistant. Wallets with high-level access to the protocol should be cold-wallets with their private key never exposed to the internet or multi-sigs, whereas private-keys of wallets with lower-level access might be used in scripts running cron jobs for instance. Both should be as secure as possible while still achieving their necessary roles. In the current implementation, all roles besides allowedVoltageSpenders, if compromised, can cause significant or permanent damage to the protocol.

### Additional Recommendations
I would consider adding a method for the _gameServerAddress to update multiple battle records at a time. This can be done through a multi-call process or a custom implementation where the _gameServerAddress supplies equal length arrays of the variables in updateBattleRecord which iteratively calls the aforementioned function repeatedly. The current design has the _gameServerAddress call the updateBattleRecord function seemingly twice for each match (winner and loser), which quickly becomes infeasible as the number of matches in any given time period increases. The base gas costs of transactions will quickly scale the gas consumed, and overall, it doesn't seem feasible long term.

The current method of adding new gameItems is interesting and works reasonably well for "cosmetic" items requiring the items to be owned by an account for their effects to be experienced. However, for consumable-type items additional logic must be present in the smart contracts making them less feasible. The current implementation only considers the battery item, with the item's logic implementation found in the VoltageManager contract. I would encourage the project to plan ahead for what items may be added to the game ahead of time and consider which aspects of the code the items may interact with. 

While not exactly an error in the code, a highly viable strategy seems to be to play the game without staking Neuron until the player generates enough points to comfortably stake their Neuron without risking losing any until their points buffer is exhausted. There is no penalty for losing while having no points and no Neuron staked. As there is a condition for winning a match with no Neuron staked, it is feasible that players will lose while having no points and no Neuron staked. A user can exploit this by repeatedly playing the game until they accumulate a significant point buffer before staking their Neuron tokens. 

### Time spent:
15 hours