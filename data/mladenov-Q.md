# [L-01] Neuron token is using deprecated function of AccessControl

### Description
According to OpenZeppelin's documentation, the _setupRole function should only be used in the constructor, and it is also deprecated.
Reference to docs:

https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_setupRole-bytes32-address-

### Context
[Neuron.sol#L95](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L95)
[Neuron.sol#L103](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L103)
[Neuron.sol#L111](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L111)

### Recommendtaion
Use `_grantRole` function ustead `_setupRole`

## Tools used
- Manual Review