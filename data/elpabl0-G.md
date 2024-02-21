## [G-1] Redundant `AiArenaHelper.sol::addAttributeProbabilities` Call in `Constructor` Increases Gas Usage

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L41C1-L52C7

The deployment cost with the `addAttributeProbabilities` call is `1,741,279`, while the cost without this call is `1,660,491`. Therefore, it's recommended to remove this function call to reduce gas usage. Instead, consider adding a `require` statement to validate the input.

```diff
       constructor(uint8[][] memory probabilities) {
+       require(probabilities.length == 6, "Invalid number of attribute arrays");
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

## [G-2] Consider Using `address(_stakeAtRiskInstance)` Instead of Storing the Address in Storage

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L68C1-L69C33

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L192C1-L196C6

Consider using `address(_stakeAtRiskInstance)` to refer to address of the instance instead of storing the address in a storage variable. This can help to reduce gas costs.


```diff
    function setStakeAtRiskAddress(address stakeAtRiskAddress) external {
        require(msg.sender == _ownerAddress);
-        _stakeAtRiskAddress = stakeAtRiskAddress;
        _stakeAtRiskInstance = StakeAtRisk(_stakeAtRiskAddress);
    }

```