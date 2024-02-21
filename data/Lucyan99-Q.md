### [GAS-1]  Duplicate execution of the same code

You shouldn't call `addAttributeProbabilities(0, probabilities)` in the constructor on line 45, because you add the attribute probabilities manually on line 49:

*Instances (1)*:

```solidity
File: src/AiArenaHelper.sol.sol

41:    constructor(uint8[][] memory probabilities) {
42:        _ownerAddress = msg.sender;
43:
44:        // Initialize the probabilities for each attribute
45:        addAttributeProbabilities(0, probabilities);
46:
47:        uint256 attributesLength = attributes.length;
48:        for (uint8 i = 0; i < attributesLength; i++) {
49:            attributeProbabilities[0][attributes[i]] = probabilities[i];
50:            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
51:        }
52:    }

```
```solidity

131:    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
132:        require(msg.sender == _ownerAddress);
133:        require(probabilities.length == 6, "Invalid number of attribute arrays");
134:
135:        uint256 attributesLength = attributes.length;
136:        for (uint8 i = 0; i < attributesLength; i++) {
137:            attributeProbabilities[generation][attributes[i]] = probabilities[i];
138:        }
139:    }

```

[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol)

### [NC-1]  The owner of the contract can be different from the deployer.

As mention in the natspec, the `ownerAddress` parameter you send to the `constructor` function should be the address of the contract deployer. In order to make sure you don't send a wrong address when you deploy the contract, you can simply use `msg.sender` instead of the `ownerAddress` parameter.

*Instances (5)*:

```solidity
File: src/FighterFarm.sol

101:    /// @param ownerAddress Address of contract deployer.
102:    /// @param delegatedAddress Address of delegated signer for messages.
103:    /// @param treasuryAddress_ Community treasury address.
104:    constructor(address ownerAddress, address delegatedAddress, address treasuryAddress_)
105:        ERC721("AI Arena Fighter", "FTR")
106:    {
107:        _ownerAddress = ownerAddress;

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol)

```solidity
File: src/GameItems.sol

93:    /// @param ownerAddress Address of contract deployer.
94:    /// @param treasuryAddress_ Address of admin signer for messages.
95:    constructor(address ownerAddress, address treasuryAddress_) ERC1155("https://ipfs.io/ipfs/") {
96:        _ownerAddress = ownerAddress;

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol)

```solidity
File: src/MergingPool.sol

68:    /// @param ownerAddress Address of contract deployer.
69:    /// @param rankedBattleAddress Address of ranked battle contract.
70:    /// @param fighterFarmAddress Address of fighter farm contract.
71:    constructor(
72:        address ownerAddress, 
73:        address rankedBattleAddress, 
74:        address fighterFarmAddress
75:    ) {
76:        _ownerAddress = ownerAddress;

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol)

```solidity
File: src/Neuron.sol

64:    /// @param ownerAddress The address of the owner who deploys the contract
65:    /// @param treasuryAddress_ The address of the community treasury
66:    /// @param contributorAddress The address that will distribute NRNs to early contributors when 
67:    /// the initial supply is minted.
68:    constructor(address ownerAddress, address treasuryAddress_, address contributorAddress)
69:        ERC20("Neuron", "NRN")
70:    {
71:        _ownerAddress = ownerAddress;

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol)

```solidity
File: src/RankedBattle.sol

142:    /// @param ownerAddress Address of contract deployer.
143:    /// @param gameServerAddress The game server address.
144:    /// @param fighterFarmAddress Address of the FighterFarm contract.
145:    /// @param voltageManagerAddress Address of the VoltageManager contract.
146:    constructor(
147:      address ownerAddress, 
148:      address gameServerAddress,
149:      address fighterFarmAddress,
150:      address voltageManagerAddress
151:    ) {
152:        _ownerAddress = ownerAddress;

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol)
