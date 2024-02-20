# [01] Lack of deactivating functionality in the protocol.

## Impact
In the contracts `FighterFarm.sol` and `Neuron.sol`, a role-based access control system is implemented. While the owner possesses the ability to grant roles to specific addresses, an important observation is that there is currently no provision for revoking these roles. This absence of revocation capability introduces the potential for unforeseen consequences, including a lack of effective management and control over designated roles. 

## Lines of code
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L139-L142
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L93-L112

## Recommended Mitigation Steps
Consider adding functionality for deactivating roles.

# [02] Redundant checks
## Impact
These checks have no practical use because a lack of them will not change the protocol flow.

## Lines of code
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L154
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L158-L161
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L164-L165
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L139-L142
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L197-L200

