### Unnecessary duplicate initialization of a storage variable

When deploying the `AiArenaHelper.sol` smart contract, there is an unnecessary  duplicate initialization with same values for the storage variable `attributeProbabilities`. The L49 should be removed because the storage variable has been already initialzied with the same values by the function `addAttributeProbabilities()` at L45.

```solidity
File: src/AiArenaHelper.sol

49       attributeProbabilities[0][attributes[i]] = probabilities[i];

137      attributeProbabilities[generation][attributes[i]] = probabilities[i];

```