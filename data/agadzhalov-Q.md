# [QA-1] `attributeProbabilities` is updated twice 
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L47

## Summary
In constructor `attributeProbabilities` are updated twice. Once by calling `addAttributeProbabilities(0, probabilities)` and once by implementing for loop.

## Details
```
constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
->        addAttributeProbabilities(0, probabilities); // first update is here

        uint256 attributesLength = attributes.length;
->        for (uint8 i = 0; i < attributesLength; i++) { // second update is here
            attributeProbabilities[0][attributes[i]] = probabilities[i];
->            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i]; // this is missed in addAttributeProbabilities()
        }
    } 
```

## Recommended Mitigation Steps
Update `attributeProbabilities` only once by calling `addAttributeProbabilities(0, probabilities);` and most likely also implementing `attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i]`  in `addAttributeProbabilities()`

# [QA-2] `attributesLength` is `uint256` and `i` is `uint8`

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L73

## Summary
`attributesLength` is from type `uint256` and `i` is from type `uint`8.

## Details
```
function addAttributeDivisor(uint8[] memory attributeDivisors) external {
        require(msg.sender == _ownerAddress);
        require(attributeDivisors.length == attributes.length);

->         uint256 attributesLength = attributes.length;
->         for (uint8 i = 0; i < attributesLength; i++) {
            attributeToDnaDivisor[attributes[i]] = attributeDivisors[i];
        }
    }  
```

## PoC
```
function testArr() public view {
        uint256 l = 256;
        vm.expectRevert()
        for(uint8 i=0; i < l; i++) {}
    }
```

## Impact
In case `attributesLength` is bigger than `255` the method will revert

## Recommended Mitigation Steps
Keep `attributesLength` and `i` from the same type or use casting.

# [QA-3] Type used for `generation` is not consistent

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L85
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L144

## Summary
`generation` is set to be of type `uint256` and there are multiple instances where `generation` is used as `uint8`
```
/// @notice Mapping tracking fighter generation to attribute probabilities
mapping(uint256 => mapping(string => uint8[])) public attributeProbabilities;
```
## PoC
```
-> function deleteAttributeProbabilities(uint8 generation) public { //@audit-issue QA generation here is also uint8
        require(msg.sender == _ownerAddress); //@ok

        uint256 attributesLength = attributes.length; //@ok
        for (uint8 i = 0; i < attributesLength; i++) { //@ok
            attributeProbabilities[generation][attributes[i]] = new uint8[](0);
        }
    }
```
It doesn't make sense to have `generation` of uint256 and then pass only `uint8` to get `attributeProbabilities[generation]`. 

## Recommended Mitigation Steps
Keep `generation` from the same type or use casting.

# [QA-4] Passing an array of `uint256` to a loop iterating only to `type(uint16).max`

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L234
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L249

## Summary 
Array of is passed to In case the array is bigger consists of elements more than `type(uint16).max` method would revert.
## Impact
Revert

# [QA-5] Misleading comment

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L50

## Summary
Mapping is not of `address` but of `tokenId`.
```
-> /// @notice Mapping of address to fighter points.
mapping(uint256 => uint256) public fighterPoints;
```

## Recommended Mitigation Steps
Changed comment
Mapping of `address` to fighter points. -> Mapping of `tokenId` to fighter points.

# [QA-5] Refactor comment docs for `getUnclaimedRewards`

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L169

## Summary
It's not getting the unclaimed `rewards`, but it gets the `amount` (uint256) of unclaimed rewards. Also the method is `view` and users can mistake it for a method that changes the state.
```
->  /// @notice Gets the unclaimed rewards for a specific address.
    /// @param claimer The address of the claimer.
    /// @return numRewards The amount of unclaimed fighters.
->    function getUnclaimedRewards(address claimer) external view returns(uint256) {
        uint256 winnersLength;
        uint256 numRewards = 0;
        uint32 lowerBound = numRoundsClaimed[claimer];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            winnersLength = winnerAddresses[currentRound].length;
            for (uint32 j = 0; j < winnersLength; j++) {
                if (claimer == winnerAddresses[currentRound][j]) {
                    numRewards += 1;
                }
            }
        }
        return numRewards;
    }
```

## Recommended Mitigation Steps
Use better naming for both method name and refactor comment to be more meaninfut that actually we retrieve the number of unclaimed rewards.

# [QA-6] Functionality doesn't work as explained docs/comments

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L62

## Summary 
This comment is wrong. The roles are not granted by the `constructor` but by methods like `addMinter`, `addStaker`, `addSpender` etc
```
->  /// @notice Grants roles to the ranked battle contract. 
    /// @notice Mints the initial supply of tokens.
    /// @param ownerAddress The address of the owner who deploys the contract
    /// @param treasuryAddress_ The address of the community treasury
    /// @param contributorAddress The address that will distribute NRNs to early contributors when 
    /// the initial supply is minted.
    constructor(address ownerAddress, address treasuryAddress_, address contributorAddress)
        ERC20("Neuron", "NRN")
    {
        _ownerAddress = ownerAddress;
        treasuryAddress = treasuryAddress_;
        isAdmin[_ownerAddress] = true;
        _mint(treasuryAddress, INITIAL_TREASURY_MINT);
        _mint(contributorAddress, INITIAL_CONTRIBUTOR_MINT);
    }
```
## Recommended Mitigation Steps
Refactor comment.