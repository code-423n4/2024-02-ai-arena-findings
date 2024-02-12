#### 1. TransferOwnership without accept

##### Description
For the contract owner:
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/VoltageManager.sol#L64

##### Recommendation
We recommend implementing a 2-step ownership transfer pattern.