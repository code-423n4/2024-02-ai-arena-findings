# [L-1] The `safeTransferFrom` function checks `transferabl` before transferring, but doesn't revert if transfer fails due to other reasons.

Lines of code: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L291-L303

Recommendation: Consider adding appropriate error handling.

# [L-2]  The `createGameItem` function checks `isAdmin` but relies on `msg.sender` being the contract deployer. 

Lines of code: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L219

Recommendation: Consider explicitly verifying `msg.sender == _ownerAddress` for this function if that's the intended behavior.

# [L-3] Lack of two-step verification for critical functions

Lines of code: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L120-L123 , https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L167 , https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L108

Recommendation: Relying solely on `msg.sender` verification might be vulnerable if the owner's address is compromised. Consider using signed messages or multi-signature wallets for critical functions like transferring ownership or treasury address.

# [L-4]  Functions interacting with external contracts `(e.g., neuronInstance)` should be designed to prevent reentrancy vulnerabilities.

Even if the function follows the best practice of check-effects-interaction, not using a reentrancy guard when there may be transfer hooks will open the users of this protocol up to read-only reentrancies with no way to protect against it, except by block-listing the whole protocol.

Lines of code: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L375-L376

Recommendation: Use a nonreentrant modifier for this external call functions.

# [L-5]  Use of `ecrecover` is susceptible to signature malleability

The built-in EVM precompile `ecrecover` is susceptible to signature malleability, which could lead to replay attacks. References: https://swcregistry.io/docs/SWC-117 , https://swcregistry.io/docs/SWC-121 , https://medium.com/cryptronics/signature-replay-vulnerabilities-in-smart-contracts-3b6f7596df57
While this is not immediately exploitable, this may become a vulnerability if used elsewhere.

Lines of code: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Verification.sol#L40

Recommendation: Avoid using `ecrecover()` for signature verification. Instead, utilize the `OpenZeppelin's` latest version of ECDSA to ensure signatures are safe from malleability issues.

# [N-1]  Functions like `reRoll` don't verify token approval before transferring `rerollCost`.

Lines of code: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L370
Recommendation: Implement proper approval checks.

# [N-2] Ensure the `keccak256` usage for generating random values is implemented securely and unpredictably.

Lines of code: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L379
Recommendation: Consider using dedicated randomness oracles.

# [N-3] In `stakeNRN`, the call to `_getStakingFactor` might lead to stack overflow errors for deeply nested calls (if `_getStakingFactor` itself makes external calls).

Lines of code: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L258
Recommendation: Consider optimizing function calls or using alternative data structures.

# [N-4] Both `stakeNRN` and `unstakeNRN` update `_calculatedStakingFactor` even if it's already `true`. This might be unnecessary and could be optimized.
Lines of code: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L262 , https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L281

# [N-5] Error handling

While some checks throw errors `(e.g., require)`, consider handling any potential failures during operations like `_neuronInstance.transferFrom` or `_mint`. This could involve returning appropriate error messages or reverting state changes.
Lines of code: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L376-L391

# [N-6] Event data

Events like `BoughtItem` could include more detailed information like item name or price.
Lines of code: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L174

