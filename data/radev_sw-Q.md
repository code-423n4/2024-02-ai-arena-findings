# QA Report - AI Arena Audit Contest

### Findings Summary

| ID | Title | Impact | Instances |
| --- | --- | --- | --- |
| 01 | Unfair Voltage calculation in `VoltageManager.sol#spendVoltage()`. Improve Fairness in Voltage Deduction Logic | Medium | Low |
| 02 | Centralization Risk in `FighterFarm.sol#setTokenURI()` | Medium | Low |
| 03 | Inconsistent Ownership Initialization Across Contracts | Low | Low |
| 04 | Ownership Renouncement Risk in `FighterFarm.sol` contract | Medium | Low |
| 05 | Misleading Comments and Potential Exploits in `Neuron.sol` | Low | Low |
| 06 | Missing `VoltageRemaining` Event Emission in `VoltageManager.sol#_replenishVoltage()` | Low | Low |
| 07 | Hardcoded `tokenId` in `VoltageManager.sol#useVoltageBattery()` | Low | Low |
| 08 | Inefficient `remainingSupply()` function in `GameItems.sol` | Low | Low |
| 09 | Dependency on `fighterFarmContractAddress` in `AAMintPass.sol` | Medium | Low |
| 10 | `FighterFarm.sol#redeemMintPass()` issues | Medium | Low |
| 11 | Replace direct usage of `ecrecover` in `Verification.sol#verify()` with `OpenZeppelin's ECDSA library` for enhanced security and readability. Also implement deadline for the signature used in `FighterFarm.sol#claimFighters()` function | High | Low |
| 12 | Users can increase `totalNumTrained` and `numTrained` of their tokens via `FighterFarm.sol#updateModel()` as many times as they want without actually update the `NFT` | Medium | Low |
| 13 | JSON Injection and Cross-Site Scripting Attacks | High | Low |
| 14 | Create `onlyOwner` modifier for all Ownership Checks Across Contracts | NC | NC |
| 15 | Incorrect Comment for `MergingPool.sol#fighterPoints()` function | NC | NC |
| 16 | Missing `Unlocked` Event in `GameItems.sol#createGameItem()` | NC | NC |
| 17 | Events Should Be Emitted Before External Calls | NC | NC |
| 18 | `Non-external`/`public` State Variables Should Begin with an Underscore | NC | NC |
| 19 | Prefer Skip Over Revert Model in Iteration | NC | NC |
| 20 | Redundant Inheritance Specifier | NC | NC |
| 21 | Use of Struct to Encapsulate Multiple Function Parameters | NC | NC |

#

#

### 1: Unfair Voltage Calculation/Set in `VoltageManager.sol#spendVoltage()`. Improve Fairness in Voltage Deduction Logic

#### Lines of Code

[`VoltageManager.sol#spendVoltage()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol)

#### Description and Impact

The current implementation of the `spendVoltage()` function in `VoltageManager.sol` may lead to unfair scenarios where a user's existing voltage (`ownerVoltage[spender]`) is not optimally utilized before replenishing to the maximum voltage capacity (e.g., 100). This approach can result in inefficient and potentially unfair usage of the voltage, especially in cases where a user's existing voltage could cover some or all of the `voltageSpent`, thereby preserving the need for a full replenishment.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Adjust the `spendVoltage()` function to first deduct the `voltageSpent` from the user's existing `ownerVoltage[spender]`. If the user's existing voltage is sufficient to cover the `voltageSpent`, there should be no need to call `_replenishVoltage()`. Only when the existing voltage is insufficient should `_replenishVoltage()` be called, and it should replenish only the necessary amount to cover the deficit, rather than resetting to the maximum capacity. This ensures a more equitable and efficient use of voltage.

*Example Mitigation/Implementation:*

```solidity
    function spendVoltage(address spender, uint8 voltageSpent) public {
        require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);

        if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
            voltageSpend = voltageSpend - ownerVoltage[spender]; // +++
            _replenishVoltage(spender);
        }

        ownerVoltage[spender] -= voltageSpent;
        emit VoltageRemaining(spender, ownerVoltage[spender]);
    }

    /// @notice Replenishes voltage and sets the replenish time to 1 day from now
    /// @dev This function is called internally to replenish the voltage for the owner.
    /// @param owner The address of the owner
    function _replenishVoltage(address owner) private {
        ownerVoltage[owner] = 100;
        ownerVoltageReplenishTime[owner] = uint32(block.timestamp + 1 days);
    }
```

In this revised implementation, when `ownerVoltageReplenishTime[spender] <= block.timestamp`, `voltageSpend` is reduced as much as it can using the spender's existing voltage.

### 2: Centralization Risk in `FighterFarm.sol#setTokenURI()`

#### Lines of Code

[`FighterFarm.sol#setTokenURI()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol)

#### Description and Impact

The `setTokenURI()` function in `FighterFarm.sol` presents a centralization risk, as it allows the contract owner or a designated authority to alter the token URI of NFTs post-minting. This capability could be used to change the metadata of NFTs, potentially affecting their appearance, attributes, or other characteristics that are supposed to be immutable once minted. Centralization risks in decentralized applications (dApps) can lead to trust issues, as users might be concerned about the potential for misuse or arbitrary changes to their assets.

[GitHub Issue Link](https://github.com/code-423n4/2022-02-skale-findings/issues/26)

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Implement a decentralized governance mechanism for any changes to NFT metadata, ensuring that no single party has the authority to alter token URIs post-minting. This could involve community votes or a multi-signature wallet controlled by several trusted community members. Additionally, consider making NFT metadata immutable after minting, if feasible for the application's use case.

---

### 3: Inconsistent Ownership Initialization Across Contracts

#### Lines of Code

[`FighterFarm.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol), [`Neuron.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol), [`MergingPool.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol), [`VoltageManager.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol), [`GameItems.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol), [`RankedBattle.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol) constructors

#### Description and Impact

The constructors of several contracts accept an `ownerAddress` parameter to set the contract's owner, which contradicts the documentation stating that the contract deployer is the initial owner. This inconsistency can lead to confusion and potential security risks if the deployer's intention is to retain ownership but mistakenly configures the contracts with a different address.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Clarify the intended ownership model in the documentation and code comments. If the deployer is meant to be the owner, remove the `ownerAddress` parameter from the constructors and set the owner to `msg.sender`. If passing ownership at deployment is desired, ensure this is clearly documented and understood by users and developers.

---

### 4: Ownership Renouncement Risk in `FighterFarm.sol`

#### Lines of Code

[`FighterFarm.sol#transferOwnership()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol)

#### Description and Impact

The `transferOwnership()` function in `FighterFarm.sol` lacks a check for the zero address, allowing the contract owner to renounce ownership by transferring it to the zero address. This action is irreversible and could result in a loss of administrative control over the contract, potentially freezing critical functionalities that require owner permissions.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Prevent the transfer of ownership to the zero address by adding a require statement that checks if the new owner address is not the zero address. This ensures that ownership can only be transferred to a valid address, mitigating the risk of accidental or intentional renouncement.

---

### 5: Misleading Comments and Potential Exploits in `Neuron.sol`

#### Lines of Code

[`Neuron.sol#approveStaker()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol), [`Neuron.sol#approveSpender()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol)

#### Description and Impact

The comments in `Neuron.sol` regarding the `approveStaker()` function are misleading, as the function actually approves a specified `spender`, not a staker. This discrepancy can cause confusion about the function's purpose and behavior. Additionally, the `approveSpender()` function's logic, which approves `msg.sender` as a spender, may be exploited by malicious actors, posing at least a low-severity security risk.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Clarify the function comments in `Neuron.sol` to accurately reflect their behavior and parameters. Review and potentially redesign the approval mechanism in `approveSpender()` to prevent misuse, ensuring that only authorized addresses can be approved as spenders. Implementing checks or additional controls on who can call these functions may mitigate potential exploits.

---

### 6: Missing `VoltageRemaining` Event Emission in `VoltageManager.sol#_replenishVoltage()`

#### Lines of Code

[`VoltageManager.sol#_replenishVoltage()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol)

#### Description and Impact

The `_replenishVoltage()` function in `VoltageManager.sol` does not emit a `VoltageRemaining` event upon replenishing a user's voltage. This omission can reduce transparency and hinder tracking of voltage replenishments in the system, affecting user trust and the ability to audit interactions with the contract.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Modify the `_replenishVoltage()` function to emit a `VoltageRemaining` event after successfully replenishing voltage. This will enhance transparency and provide a reliable way for users and external systems to monitor voltage replenishments.

---

### 7: Hardcoded `tokenId` in `VoltageManager.sol#useVoltageBattery()`

#### Lines of Code

[`VoltageManager.sol#useVoltageBattery()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol)

#### Description and Impact

The `useVoltageBattery()` function in `VoltageManager.sol` uses a hardcoded `tokenId` for burning game item tokens, which reduces flexibility and could lead to errors if the tokenId for the battery item changes. This design choice limits the contract's adaptability to changes in game items.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Replace the hardcoded `tokenId` with a state variable that can be set and updated through a function accessible only by the contract owner or through a decentralized governance mechanism. This will increase the contract's flexibility and adaptability to changes in game items.

---

### 8: Inefficient `remainingSupply()` function in `GameItems.sol`

#### Lines of Code

[`GameItems.sol#remainingSupply()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol)

#### Description and Impact

The current implementation of the `remainingSupply()` function in `GameItems.sol` does not first check if the supply of a specific `tokenId` (NFT) is finite before returning the `itemsRemaining` parameter. This oversight could lead to incorrect supply calculations for infinite-supply items, potentially causing confusion or errors in supply management.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Revise the `remainingSupply()` function to include a check for the finiteness of a token's supply before returning the `itemsRemaining` parameter. This adjustment will ensure accurate supply calculations and improve the function's reliability and accuracy.

---

### 9: Dependency on `fighterFarmContractAddress` in `AAMintPass.sol`

#### Lines of Code

[`FighterFarm.sol#redeemMintPass()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol), [`AAMintPass.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AAMintPass.sol)

#### Description and Impact

The `redeemMintPass()` function in `FighterFarm.sol` will always revert unless the `fighterFarmContractAddress` is set in the `AAMintPass.sol` contract. This dependency creates a potential point of failure, as the functionality of redeeming mint passes is contingent upon the correct configuration of the `fighterFarmContractAddress`.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

To mitigate this issue, consider setting the `fighterFarmContractAddress` in the constructor of the `AAMintPass.sol` contract or develop a deployment script that immediately sets the `fighterFarmContractAddress` in `AAMintPass.sol` after deploying the `FighterFarm.sol` contract. This approach will ensure that the necessary configuration is in place for the `redeemMintPass()` function to operate correctly.

---

### 10: `FighterFarm.sol#redeemMintPass()` issues

#### Lines of Code

[`FighterFarm.sol#redeemMintPass()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol)

#### Description and Impact

The `redeemMintPass()` function in `FighterFarm.sol` does not perform a check on the length of the `iconsTypes[]` array, potentially leading to function reversion if the array length does not meet expected criteria. Additionally, the method for generating `dna` for fighters (`uint256(keccak256(abi.encode(mintPassDnas[i])))`) does not prevent the possibility of multiple fighters having the same `dna`, which could lead to issues with fighter uniqueness and integrity.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Implement a validation check for the `iconsTypes[]` array length to ensure it meets the required criteria before proceeding with the rest of the function's logic. To address the issue of potential `dna` collisions, consider introducing additional unique parameters to the `abi.encode` function call or implementing a collision detection mechanism to ensure each fighter's `dna` is unique.

---

### 11: Replace direct usage of `ecrecover` in `Verification.sol#verify()` with `OpenZeppelin's ECDSA library` for enhanced security and readability. Also implement deadline for the signature used in `FighterFarm.sol#claimFighters()` function

#### Lines of Code

[`Verification.sol#verify()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Verification.sol), [`FighterFarm.sol#claimFighters()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol)

#### Description and Impact

Direct usage of `ecrecover` in `Verification.sol#verify()` could be replaced with OpenZeppelin's `ECDSA` library to enhance security and readability. Additionally, the absence of a deadline for signatures used in `FighterFarm.sol#claimFighters()` could expose the function to replay attacks.

[Signature Attacks Reference](https://scsfg.io/hackers/signature-attacks/#missing-validation)

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Refactor the `verify()` function to utilize OpenZeppelin's `ECDSA` library for signature recovery, which provides additional validation checks and improves code readability. Introduce a signature deadline mechanism in `claimFighters()` and ensure that each signature can only be used within a specified timeframe to mitigate the risk of replay attacks.

---

### 12: Users can increase `totalNumTrained` and `numTrained` of their tokens via `FighterFarm.sol#updateModel()` as many times as they want without actually update the `NFT`

#### Lines of Code

[`FighterFarm.sol#updateModel()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol)

#### Description and Impact

Users can exploit the `updateModel()` function in `FighterFarm.sol` to artificially increase the `totalNumTrained` and `numTrained` counters for their tokens without actually updating the NFT. This could undermine the integrity of the training system and impact the game's balance.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Implement checks within the `updateModel()` function to verify that a meaningful update (`modelHash` and `modelType` are different) to the NFT has occurred before incrementing the `totalNumTrained` and `numTrained` counters. This could involve validating changes to the model or other relevant attributes associated with the NFT.

---

### 13: JSON Injection and Cross-Site Scripting Attacks

#### Lines of Code

[`FighterFarm.sol#setTokenURI()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol), [`GameItems.sol#createGameItem()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol), [`GameItems.sol#setTokenURI()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol)

#### Description and Impact

The `setTokenURI()` function in `FighterFarm.sol` and `GameItems.sol`, along with the `createGameItem()` function in `GameItems.sol`, are vulnerable to JSON injection and Cross-Site Scripting (XSS) attacks. These vulnerabilities could allow attackers to inject malicious code or manipulate the JSON metadata associated with NFTs, potentially leading to security breaches or manipulation of displayed content.

[JSON Injection Example 1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/167), [XSS Attack Example 2](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/53)

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Sanitize and validate all inputs to the `setTokenURI()` and `createGameItem()` functions to prevent injection attacks. Consider using established libraries or frameworks for handling user inputs and generating JSON metadata. Additionally, implement content security policies (CSP) to mitigate the impact of any potential XSS vulnerabilities.

### 14: Create `onlyOwner` modifier for all Ownership Checks Across Contracts

#### Lines of Code

Multiple functions in [`FighterFarm.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol), [`Neuron.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol), [`MergingPool.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol), [`VoltageManager.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol), [`GameItems.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol), [`RankedBattle.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol)

#### Description and Impact

Several contracts in the AI Arena Protocol perform redundant checks for `msg.sender` being the owner in functions restricted to the owner. This redundancy increases the complexity and gas cost of these functions. Implementing a unified `onlyOwner` modifier would streamline the code, reduce gas costs, and decrease the likelihood of errors in future development.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Define an `onlyOwner` modifier in each contract that requires ownership verification. Replace individual `msg.sender` ownership checks with this modifier to simplify the code and enhance security by centralizing the ownership logic.

---

### 15: Incorrect Comment for `MergingPool.sol#fighterPoints`

#### Lines of Code

[`MergingPool.sol#fighterPoints`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol)

#### Description and Impact

The comment for the `fighterPoints` state variable in `MergingPool.sol` inaccurately describes its purpose. It states that it maps an address to fighter points, whereas it actually maps a tokenId to fighter points. This discrepancy can lead to confusion and misunderstanding about how fighter points are tracked and associated with tokens rather than user addresses.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Correct the comment to accurately reflect that `fighterPoints` maps a tokenId to its corresponding fighter points. This will improve code readability and maintainability by ensuring that comments accurately describe the code's functionality.

---

### 16: Missing `Unlocked` Event in `GameItems.sol#createGameItem()`

#### Lines of Code

[`GameItems.sol#createGameItem()`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol)

#### Description and Impact

The `createGameItem()` function in `GameItems.sol` does not emit an `Unlocked` event, which could be useful for tracking the creation and unlocking of new game items. This omission may reduce the visibility of such events, impacting the ability of users and external systems to monitor game item dynamics.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Enhance the `createGameItem()` function by emitting an `Unlocked` event upon the successful creation of a new game item. This will improve transparency and enable better tracking and verification of game item creation events.

---

### 17: Events Should Be Emitted Before External Calls

#### Lines of Code

Various instances across multiple files:
- [GameItems.sol Lines 174, 231](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol/#L174-L174)
- [src/MergingPool.sol Line 165](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol/#L165-L165)

#### Description and Impact

Events are not consistently emitted before external calls across the codebase, potentially leading to issues with the check-effects-interactions pattern. This practice is crucial for ensuring that state changes are logged before any interaction with external contracts, which can help prevent reentrancy attacks and other vulnerabilities.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Refactor the code to ensure that all events are emitted immediately after state changes and before any external calls. This approach adheres to the check-effects-interactions pattern, enhancing the contract's security and reliability.

**Example Refactor:**

```solidity
// Before external call
emit EventName(parameters);

// External call
externalContract.call(parameters);
```

Ensure this pattern is consistently applied across all contracts to maintain the integrity and security of the system.

---

### 18: `Non-external`/`public` State Variables Should Begin with an Underscore

#### Lines of Code

Instances in [src/GameItems.sol Lines 36-41](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol/#L36-L41) and [src/RankedBattle.sol Lines 43-45](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol/#L43-L45).

#### Description and Impact

Non-`external`/`public` state variables not beginning with an underscore deviate from the Solidity Style Guide, potentially leading to confusion regarding their visibility and use within the contract.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Refactor the state variable names to begin with an underscore, aligning with the Solidity Style Guide and improving code readability and consistency.

**Before:**

```solidity
uint256 itemsRemaining;
```

**After:**

```solidity
uint256 _itemsRemaining;
```

This naming convention clearly differentiates between external/public and internal/private variables, enhancing code clarity.

---

### 19: Prefer Skip Over Revert Model in Iteration

#### Lines of Code

[FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol/#L251-L251)

#### Description and Impact

Reverting transactions in loops based on conditional checks can lead to denial of service, especially if malicious actors can influence the conditions to consistently fail. Preferring a skip-over model enhances robustness and prevents potential manipulation.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Implement a skip-over logic where conditions that would otherwise cause a revert instead skip the current iteration, allowing the loop to continue processing other elements.

**Before:**

```solidity
for (uint16 i = 0; i < arrayLength; i++) {
    require(condition, "Revert condition met");
    // Process array element
}
```

**After:**

```solidity
for (uint16 i = 0; i < arrayLength; i++) {
    if (!condition) {
        continue; // Skip to the next iteration if condition is not met
    }
    // Process array element
}
```

This change ensures that the function remains resilient against inputs that could otherwise cause it to revert, improving the contract's overall reliability.

---

### 20: Redundant Inheritance Specifier

#### Lines of Code

[FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol/#L16-L16)

#### Description and Impact

The `FighterFarm` contract unnecessarily specifies `ERC721` in its inheritance list despite `ERC721Enumerable` already extending `ERC721`. This redundancy does not impact the contract's functionality but goes against best practices for clean and efficient code.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Remove the redundant `ERC721` from the inheritance list of the `FighterFarm` contract to streamline the code and adhere to best practices.

**Before:**

```solidity
contract FighterFarm is ERC721, ERC721Enumerable {
```

**After:**

```solidity
contract FighterFarm is ERC721Enumerable {
```

This change simplifies the inheritance chain and clarifies the contract's structure.

---

### 21: Use of Struct to Encapsulate Multiple Function Parameters

#### Lines of Code

[FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol/#L235-L244)
[GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol/#L208-L218)
[RankedBattle.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol/#L322-L330)

#### Description and Impact

Functions with a high number of parameters can lead to decreased readability and increased complexity, making the code harder to maintain and more prone to errors. Encapsulating parameters in a struct can significantly improve code clarity and reusability.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Refactor the functions by defining a struct that encapsulates the parameters and passing an instance of this struct instead. This approach simplifies the function signatures and enhances code readability.

**Example Refactor for `redeemMintPass`:**

**Before:**

```solidity
function redeemMintPass(
    uint256[] calldata mintpassIdsToBurn,
    uint8[] calldata fighterTypes,
    // Additional parameters
) external {
```

**After:**

```solidity
struct RedeemMintPassParams {
    uint256[] mintpassIdsToBurn;
    uint8[] fighterTypes;
    // Additional parameters
}

function redeemMintPass(RedeemMintPassParams calldata params) external {
```

Apply this pattern to all identified instances, ensuring that function calls are updated accordingly to pass a struct instance. This change not only improves the code's readability but also its maintainability and scalability.
