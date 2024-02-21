# Gas Optimization


## [G-01] Avoid Making any STATE reads if we can revert early (Save 2100 Gas)

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131-L139

```solidity
File: src/AiArenaHelper.sol

131    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
132        require(msg.sender == _ownerAddress);
133        require(probabilities.length == 6, "Invalid number of attribute arrays");
134
135        uint256 attributesLength = attributes.length;
136        for (uint8 i = 0; i < attributesLength; i++) {
137            attributeProbabilities[generation][attributes[i]] = probabilities[i];
138        }
139    }
```

Avoid reading from storage if we might revert early. The first check involves making a state read, `_ownerAddress` The second check, the require statement, checks if a function parameter `probabilities` is equal to `6` and reverts if it happens not. As it is, if `probabilities` is not 6, we would consume gas in the require statement to make the state read(`2100 Gas for the state read`) and then revert on the second check we can save some gas in the case of such a revert by moving the cheap check to the top and only reading state variable after validating the function parameter.

Also in this function

```solidity
File: src/FighterFarm.sol

191    function claimFighters(
192        uint8[2] calldata numToMint,
193        bytes calldata signature,
194        string[] calldata modelHashes,
195        string[] calldata modelTypes
196    )
197        external
198    {
199        bytes32 msgHash = bytes32(keccak256(abi.encode(
200            msg.sender,
201            numToMint[0],
202            numToMint[1],
203            nftsClaimed[msg.sender][0],
204            nftsClaimed[msg.sender][1]
205        )));
206        require(Verification.verify(msgHash, signature, _delegatedAddress));
207        uint16 totalToMint = uint16(numToMint[0] + numToMint[1]);
208        require(modelHashes.length == totalToMint && modelTypes.length == totalToMint); // @audit first check this one
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L191-L210

```solidity
File: src/Neuron.sol

127    function setupAirdrop(address[] calldata recipients, uint256[] calldata amounts) external {
128        require(isAdmin[msg.sender]);
129        require(recipients.length == amounts.length);
130        uint256 recipientsLength = recipients.length;
131        for (uint32 i = 0; i < recipientsLength; i++) {
132            _approve(treasuryAddress, recipients[i], amounts[i]);
133        }
134    }
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L127-L134

## [G‑02] Add unchecked{} for subtraction where the operands can't underflow becasue of previous IF-STATEMENT

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L201

```solidity
File: src/Neuron.sol

201        uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
```

**This will not undferflow becasue of this check**

```solidity
File: src/Neuron.sol

197        require(
198            allowance(account, msg.sender) >= amount,
199            "ERC20: burn amount exceeds allowance"
200        );
```

## [G-03] Expressions for constant values such as a call to  KECCAK256(), should use IMMUTABLE rather than CONSTANT

```solidity
File: src/Neuron.sol

28    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

31    bytes32 public constant SPENDER_ROLE = keccak256("SPENDER_ROLE");

34    bytes32 public constant STAKER_ROLE = keccak256("STAKER_ROLE");
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L28-L34

## [G-04] Pre-increment and pre-decrement are cheaper than +1 ,-1

```solidity
File: src/AiArenaHelper.sol

                attributeProbabilityIndex = i + 1;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L181

## [G-05] STATE VARIABLE should not used in emit

**missed from bot**

```solidity
File: src/GameItems.sol

231          emit Locked(_itemCount);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L231

## [G-06] Access MAPPINGS directly rather than using accessor functions

When you have a mapping, accessing its values through accessor functions involves an additional layer of indirection, which can incur some gas cost. This is because accessing a value from a mapping typically involves two steps: first, locating the key in the mapping, and second, retrieving the corresponding value.

```solidity
File: src/FighterFarm.sol

402    function tokenURI(uint256 tokenId) public view override(ERC721) returns (string memory) {
403        return _tokenURIs[tokenId];
404    }
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L402-L404

```solidity
File: src/RankedBattle.sol

379    function getNrnDistribution(uint256 roundId_) public view returns(uint256) {
380        return rankedNrnDistribution[roundId_];
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L379

```solidity
File: src/StakeAtRisk.sol

132    function getStakeAtRisk(uint256 fighterId) external view returns(uint256) {
133        return stakeAtRisk[roundId][fighterId];
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L132-L134

## [G-07] Consider using OZ EnumerateSet in place of nested mappings

Nested mappings and multi-dimensional arrays in Solidity operate through a process of double hashing, wherein the original storage slot and the first key are concatenated and hashed, and then this hash is again concatenated with the second key and hashed. This process can be quite gas expensive due to the double-hashing operation and subsequent storage operation (sstore).

A possible optimization involves manually concatenating the keys followed by a single hash operation and an sstore. However, this technique introduces the risk of storage collision, especially when there are other nested hash maps in the contract that use the same key types. Because Solidity is unaware of the number and structure of nested hash maps in a contract, it follows a conservative approach in computing the storage slot to avoid possible collisions.

OpenZeppelin's EnumerableSet provides a potential solution to this problem. It creates a data structure that combines the benefits of set operations with the ability to enumerate stored elements, which is not natively available in Solidity. EnumerableSet handles the element uniqueness internally and can therefore provide a more gas-efficient and collision-resistant alternative to nested mappings or multi-dimensional arrays in certain scenarios.

```solidity
File: src/AiArenaHelper.sol

30    mapping(uint256 => mapping(string => uint8[])) public attributeProbabilities;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L30

```solidity
File: src/FighterFarm.sol

88    mapping(address => mapping(uint8 => uint8)) public nftsClaimed;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L88

```solidity
File: src/GameItems.sol

74    mapping(address => mapping(uint256 => uint256)) public allowanceRemaining;

77    mapping(address => mapping(uint256 => uint256)) public dailyAllowanceReplenishTime;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L74-L78

```solidity
File: src/RankedBattle.sol

113    mapping(address => mapping(uint256 => uint256)) public accumulatedPointsPerAddress;

116    mapping(uint256 => mapping(uint256 => uint256)) public accumulatedPointsPerFighter;

125    mapping(uint256 => mapping(uint256 => bool)) public hasUnstaked;

134    mapping(uint256 => mapping(uint256 => bool)) _calculatedStakingFactor;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L113

```solidity
File: src/StakeAtRisk.sol

46    mapping(uint256 => mapping(uint256 => uint256)) public stakeAtRisk;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L46

## [G-08] Using STORAGE instead of MEMORY for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from
storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.

If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
File: src/AiArenaHelper.sol

96            uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributes.length);

174        uint8[] memory attrProbabilities = getAttributeProbabilities(generation, attribute);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L96

```solidity
File: src/FighterFarm.sol

510        FighterOps.FighterPhysicalAttributes memory attrs = _aiArenaHelperInstance.createPhysicalAttributes(
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L510

```solidity
File: src/MergingPool.sol

123        address[] memory currentWinnerAddresses = new address[](winnersLength);

206        uint256[] memory points = new uint256[](1);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L123

## [G-09] Using fixed BYTES CONSTANTS is cheaper than using string

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

```solidity
File: src/GameItems.sol

48    /// @notice The name of this smart contract.
49    string public name = "AI Arena Game Items";

51    /// @notice The symbol for this smart contract.
52    string public symbol = "AGI";
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L49-L53

## [G-10] Using  CALLDATA instead of  MEMORY for read-only arguments in external functions

When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 \* <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.

```solidity
File: src/AiArenaHelper.sol

131    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {

157    function getAttributeProbabilities(uint256 generation, string memory attribute)

169    function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute)

```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L169

```solidity
File: src/GameItems.sol

194    function setTokenURI(uint256 tokenId, string memory _tokenURI) public {

208    function createGameItem(
209        string memory name_,
210        string memory tokenURI,

291    function safeTransferFrom(
292        address from,
293        address to,
294        uint256 tokenId,
295        uint256 amount,
296        bytes memory data
297    )
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L194

## [G-11] Use ASSEMBLY to write address storage values

```solidity
File: src/FighterFarm.sol

48    address public treasuryAddress;

51    address _ownerAddress;

54    address _delegatedAddress;

57    address _mergingPoolAddress;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L48-L57

```solidity
File: src/RankedBattle.sol

69    address _stakeAtRiskAddress;

72    address _ownerAddress;

75    address _gameServerAddress;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L69-L75

```solidity
File: src/GameItems.sol

58    address public treasuryAddress;

61    address _ownerAddress;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L57-L62

```solidity
File: src/Neuron.sol

46    address public treasuryAddress;

49    address _ownerAddress;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L45-L49

```solidity
File: src/StakeAtRisk.sol

30    address public treasuryAddress;

33    address _rankedBattleAddress;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L33

## [G-12] Use ASSEMBLY to perform efficient back-to-back calls

```solidity
File:  src/FighterFarm.sol

250            require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));
251            _mintpassInstance.burn(mintpassIdsToBurn[i]);


373        require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");
375        _neuronInstance.approveSpender(msg.sender, rerollCost);
376        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L250-L251

```solidity
File: src/GameItems.sol

163        _neuronInstance.approveSpender(msg.sender, price);
164        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, price);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L163-L164

```solidity
File: src/RankedBattle.sol

246        require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");
254                _fighterFarmInstance.updateFighterStaking(tokenId, true);


247        require(_neuronInstance.balanceOf(msg.sender) >= amount, "Stake amount exceeds balance");
250        _neuronInstance.approveStaker(msg.sender, address(this), amount);
251        bool success = _neuronInstance.transferFrom(msg.sender, address(this), amount);

336            _voltageManagerInstance.ownerVoltageReplenishTime(fighterOwner) <= block.timestamp ||
337            _voltageManagerInstance.ownerVoltage(fighterOwner) >= VOLTAGE_COST
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L246

```solidity
File: src/VoltageManager.sol

95        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
96        _gameItemsContractInstance.burn(msg.sender, 0, 1);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L95

## [G-13] Shorten the array rather than copying to a new one

ASSEMBLY can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
File: src/AiArenaHelper.sol

96            uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributes.length);

149            attributeProbabilities[generation][attributes[i]] = new uint8[](0);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L96

```solidity
File: src/MergingPool.sol

123        address[] memory currentWinnerAddresses = new address[](winnersLength);

206        uint256[] memory points = new uint256[](1);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L123

## [G-14] Simple checks for zero uint can be done using ASSEMBLY to save gas

```solidity
File: src/RankedBattle.sol

253            if (amountStaked[tokenId] == 0) {

285            if (amountStaked[tokenId] == 0) {

440        if (battleResult == 0) {

444            if (stakeAtRisk == 0) {

506        if (battleResult == 0) {

530      if (stakingFactor_ == 0) {
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L253

## [G-15] Expensive operation inside a for loop

```solidity
File: src/MergingPool.sol

149        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
150            numRoundsClaimed[msg.sender] += 1;
151            winnersLength = winnerAddresses[currentRound].length;
152            for (uint32 j = 0; j < winnersLength; j++) {
153                if (msg.sender == winnerAddresses[currentRound][j]) {
154                    _fighterFarmInstance.mintFromMergingPool(
155                        msg.sender,
156                        modelURIs[claimIndex],
157                        modelTypes[claimIndex],
158                        customAttributes[claimIndex]
159                    );
160                    claimIndex += 1;
161                }
162            }
163        }
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L149-L163

```solidity
File: src/RankedBattle.sol

299        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
300            nrnDistribution = getNrnDistribution(currentRound);
301            claimableNRN += (
302                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution
303            ) / totalAccumulatedPoints[currentRound];
304            numRoundsClaimed[msg.sender] += 1;
305        }
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L299-L305
