
### [Low -1]: AiArenaHelper:constructor potentially duplicates probabilities in the attributeProbabilities mapping during contract initialization.
**Description:** During contract initialization in the AiArenaHelper:constructor, probabilities may be inadvertently added twice to the attributeProbabilities mapping. Once via the function addAttributeProbabilities(0, probabilities) and again through the for loop.
```javascript
    constructor(uint8[][] memory probabilities) {

        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    }
```
**Recommended Mitigation:** Ensure that probabilities are added only once during contract initialization by either utilizing a mapping or employing a for loop, but not both simultaneously.
```diff
    constructor(uint8[][] memory probabilities) {

        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
-        addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    }
```

### [Low-2]: `GameItems:setTokenURI` function should be marked as private.
**Description:** This function is getting called by `createGameItem` function which creates a new game item and sets the tokenUri for that game item and increments the `_itemCount`. Any admin can call this function mistakely or intentionally and can change the token uri of any gameItem. Or admin can add a new tokenUri to `setTokenURI`  without incrementing the `_itemCount` and without changing `allGameItemAttributes`.

```javascript

   function setTokenURI(uint256 tokenId, string memory _tokenURI) public {
        require(isAdmin[msg.sender]);
        _tokenURIs[tokenId] = _tokenURI;
    }

```

### [Low-3] Wrong natspec for mapping `MergingPool.sol:fighterPoints`.
**Description:** In natspec it is written that it is the mapping of `address` to `fighter points` but the mapping is of `uint256` to `uint256(fighterPoints)`. This mapping is the mapping of `tokenId` to `fighterPoints`.
```javascript
    /// @notice Mapping of address to fighter points.
    mapping(uint256 => uint256) public fighterPoints;
```