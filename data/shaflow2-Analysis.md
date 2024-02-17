# Analysis Report
## Table of Contents
- [Analysis of the codebase](#analysis-of-the-codebase)
- [Architecture feedback](#architecture-feedback)
- [Centralization risk](#centralization-risk)
- [Systemic risks](#systemic-risks)
- [Other recommendations](#other-recommendations)
- [Audit Process](#audit-process)

## Analysis of the codebase <a id="analysis-of-the-codebase"></a>  

#### Overview  
The entire codebase predominantly utilizes imports instead of inheritance, enabling most instance contracts to change instance addresses through specific permission roles, thereby enhancing the system's upgradability. Below, I will describe some notable aspects of the code:  

* **Neuron.sol**  
  The Neuron contract serves as the native ERC20 token for the entire system, utilizing AccessControl for role management. Certain roles are granted the authority to transfer NRN tokens to any address. These roles' owners are audited contracts.  

* **AiArenaHelper.sol, MergingPool.sol, FighterFarm.sol, FighterOps.sol**  
  These contracts define the minting and circulation rules for fighter NFTs. Each minted NFT corresponds to a fighter struct. Since the contracts do not provide a burn function, minted fighters cannot be destroyed but can be transferred to other addresses under specific conditions. Additionally, the sponsors opted out of using Chainlink VRF and instead chose to control fighter attributes using predictable pseudo-random numbers during minting. When asked about this decision, the sponsors explained that true randomness is not crucial for their game as training is the most significant factor determining outcomes. They also mentioned that after testing the game for about 2.5 years on the testnet, players did not actually care about true randomness. Regarding the NFT lottery controlled by MergingPool, the contract only records points and lottery results, while the complex lottery calculation process is handled off-chain. This design is deemed reasonable as it executes only critical parts on-chain, reducing certain risks and on-chain computational burdens. However, it sacrifices some transparency in the lottery. Winners claim their NFTs by calling relevant functions themselves, rather than the contract actively distributing them, effectively preventing malicious DOS attacks.  

* **RankedBattle.sol, StakeAtRisk.sol, GameItems.sol, VoltageManager.sol**  
  These contracts govern the rules related to game battles and game items. Similar to the lottery, RankedBattle only records battle results and changes in staking status. Game outcomes are predicted using off-chain gaming oracles. GameItems utilize ERC1155, where each token corresponds to a GameItems struct, and the struct's information determines the token's supply rules.  



## Architecture feedback <a id="architecture-feedback"></a>
The entire contract system is divided into two modules: fighter acquisition (NFT minting) and staking battles.
### Fighter Acquisition:
- There are three methods to mint fighter NFTs:
  1. Minting through off-chain acquisition of signature and minting information provided by sponsors, followed by calling `claimFighters`.
  2. Minting by redeeming AAMintPass NFTs through `redeemMintPass`.
  3. Obtaining through MergingPool lottery.
- **Note: If the recipient's total number of fighters is less than the maximum quantity, and the fighters to be sent are not staked, fighters can be transferred to other addresses.**

### Battle:
  1. Administrators can advance to the next round by calling `setNewRound`, provided that the current round has accumulated points.
  2. During a round, players can stake NRN without limitation (`stakeNRN`) until they cancel their stake (`unstakeNRN`).
  3. After a battle concludes, the gameserver invokes `updateBattleRecord` twice to update information regarding both the initiator and the target of the battle. If there is no stake, it does not affect points; however, if there is a stake, it affects both points and staked assets. If the performance is poor, a portion of the assets will become "at risk," and to earn points in subsequent battles, the player must first redeem all at-risk assets through victories in battles. Only then will victories in subsequent battles increase points.
  4. After a round ends, all at-risk assets are transferred to the treasury. Owners of fighters with points can call `claimNRN` to claim NRN from the prize pool based on the total points ratio.  

**However, I discovered a high-risk vulnerability in the GameItems contract.The contract does not override the safeBatchTransferFrom function, allowing players to freely transfer their game items regardless of whether they are transferable.**
## Centralization risk <a id="centralization-risk"></a>  

Every major contract has an `_ownerAddress` to wield supreme authority, including updating protocol addresses, changing contract states, and assigning admin roles downward.  

A. `transferOwnership` is provided to transfer `_ownerAddress`, but a one-step transfer of ownership could pose risks. **A copy-paste error or a typo could potentially replace the owner with an address not controlled by the sponsor, resulting in the sponsor losing some control over the contract's permissions. This is particularly serious due to the extensive permissions held by the owner, including:**
1. Loss of the ability to update contract instance addresses, which would affect system upgrades.
2. Loss of the ability to set certain basic calculation parameters, such as `attributeDivisors` and `attributeProbabilities`.
3. Loss of the ability to update delegated permission roles, which are generally contracts necessary for inter-contract interactions, such as `adjustAllowedVoltageSpenders` in the `VoltageManager` contract. This further impacts the system's upgradability.
4. If admin permissions are not delegated in `RankedBattle`, loss of owner control could prevent rounds from progressing.  

B. **Regarding other admins below the owner:**  
   - Admins in `RankedBattle` could potentially be externally owned accounts (EOA) used to advance rounds and set prize pools.
   - Admins in `MergingPool` could potentially be EOAs used to update the number of winners per round and update the list of winners.
   - Admins in `createGameItem` could potentially be EOAs used to create and manage game items.  
**Malicious behavior by these admins could lead to:**  
1. Incorrect progression of rounds in the battle contract, such as initiating battles to gain points at the start of a new round when the total points are still low, or setting a large total prize pool maliciously to gain a large amount of NRN.
2. Malicious adjustment of the prize pool at the end of a battle round, potentially resulting in players not receiving the expected amount of NRN rewards.
3. Malicious tampering with the list of winners in the MergingPool contract.
4. Malicious creation of useless game items that cannot be revoked (as the `GameItems` struct array can only add elements and not delete them).  
Other admins or roles with special permissions should ideally be contracts within the system necessary to ensure normal functioning and interactions between different parts of the system. For instance, `adjustAllowedVoltageSpenders` in the `VoltageManager` contract should be set to the `RankedBattle` contract.  
## Systemic risks <a id="systemic-risks"></a>  

* **I believe the most significant systemic potential risk lies within Neuron, which serves as the native token of the entire system, but lacks ROLE management. Details are as follows:**  
  * In the Neuron contract, the owner can assign SPENDER_ROLE, STAKER_ROLE, and MINTER_ROLE. Among these, SPENDER_ROLE and STAKER_ROLE have immense control over tokens for any address. However, due to the absence of an ADMIN_ROLE function, these roles cannot be effectively managed. Once assigned, only the roles themselves can relinquish their permissions, and newly added contract privilege roles do not override old ones. Consequently, even _ownerAddress cannot control them to revoke permissions.  
  * Although the sponsors mentioned that the contracts granted with privilege roles are audited and trustworthy, it is well-known that even rigorously audited contracts may still contain vulnerabilities. If exploited, these vulnerabilities could have catastrophic effects on Neuron (SPENDER_ROLE and STAKER_ROLE having control over NRN tokens for any address). However, there is no method to revoke their permissions to terminate their influence on the system. Additionally, I observed that the contract roles that have been confirmed (FighterFarm.sol, GameItems.sol, RankedBattle.sol) do not encapsulate the renounceRole(bytes32 role, address account) function, so they themselves cannot relinquish ROLE privileges.
  * Consider implementing ADMIN_ROLE to address this issue.

* **Secondly, the Admin privileges at the bottom of the Battle contract are extensive and may lead to a DOS attack on the staking reward system.**  
  * Firstly, Battle mints new NRN tokens as rewards for each round. However, it is important to note that when the total supply exceeds MAX_SUPPLY, the mint function will stop minting. Therefore, an admin could maliciously set the staking rewards for a round to exceed the total supply, allowing players in that round to receive a large amount of NRN, preventing subsequent round players from successfully receiving their rewards. This situation can only be rectified by the treasury or other roles burning their NRN. Alternatively, incentivizing players who receive a large amount of NRN to destroy unlawfully obtained NRN is difficult.
  * Consider setting a maximum reward per round, which only the owner can modify, to prevent such occurrences.

* **some parameters in the contracts are hardcoded, reducing contract upgradability.** While system upgrades can be accomplished by changing contract addresses, during upgrades, not only must the contract addresses be changed, but also the settings of some contract privilege roles, which is complex and prone to oversight.
  * Consider adopting a proxy contract pattern.

* **Lastly, In GameItems contract, once set, the allowedBurningAddresses cannot be revoked.The authorized Burning Addresses have significant permissions, enabling them to potentially destroy GameItems from any address. Once an authorized contract is attacked, the destruction cannot be prevented.Please endeavor to avoid this pattern as much as possible.**  

## Other recommendations <a id="other-recommendations"></a>

* In the staking system, under similar conditions, staking amounts ranging from 1 to 1 ether yield the same number of points. For stakes below 1000 NRN, due to precision issues with division, NRN will not be at risk even when their points are 0.Additionally, regardless of how low the points are, they can offset any amount of asset loss, which could potentially be exploited. Further details are available in my other issue report.  

* Due to the omission of the NRN at risk part, certain situations may cause updateBattleRecord to revert. Further details are available in my other issue report.

* **I believe the system's management of GameItems lacks flexibility.** Once a game item is added, it cannot be deleted, and none of its attributes other than "transferable" can be modified. However, the system is subject to change, and the data set during addition, such as supply and price, may become obsolete later. Please consider allowing administrators to set new dailyAllowance, itemPrice, or even remove game items based on the system's actual needs.  


* There are some minor errors in the contracts. For instance, the view function (getFighterPoints) does not initialize the array length correctly, attributeProbabilities are initialized twice in the constructor, and totalBattle fails to accurately represent the total number of battles.  

  

## Audit Process <a id="audit-process"></a>


I spent five days auditing this code repository. With the addition of the report writing time, it may take up to five days.

I learned about the overall functionality of the project from the readme file, divided the contract into functional modules, and read the complete code in the order of Neuron -> (AiArenaHelper, FighterFarm, FighterOps) -> (GameItems, VoltageManager) -> (RankedBattle, StackAtRisk, MergingPool).

After the first reading, I marked my existing problems and some minor findings. Afterwards, I rechecked the various functions of each contract and asked sponsors some questions. Some of the questions were confirmed and written into a report.

#### Some questions I asked

1. **Question:** Hello, I have a question about why the contract abandoned VRF and used pseudo-random numbers. Is it because the importance of elements, weight, and Physical Attribute has decreased, so predicting the forging results in advance is encouraged?
   * **Answer:** Yea it’s not important in our game in terms of getting an advantage. By far the most important determinant of the outcome is training. The elements and weights simple affect the way you train and play style. Also we’ve been testing the game for about 2.5 years on testnet and players don’t actually care about true randomness.

2. **Question:** I'm wondering if I can find out how much NUN players typically stake during testing.
   * **Answer:** Up to 10k.

3. **Question:** Is it allowed for a fighter to withdraw their stake once they have achieved their satisfactory points, resulting in their points being locked in this round? In this way, they won't lose points even if they fail the attack battle.
   * **Answer:** Yes, this is allowed.

4. **Question:** I think initiating a battle, which the gameserver calls the updateBattleRecord function twice, so totalBattles will increase twice. Is this considered an issue?
   * **Answer:** Great catch - yes, this is an issue.

Thank you to @ guppy and @ Sonny for their careful answers.


### Time spent:
45 hours