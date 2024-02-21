### [G-01] Structs in `FightersOps` can be better packed to save number of SSTOREs used when setting
The structs `FighterPhysicalAttributes` and `Fighter` can both be reorded and/or use smaller uint types to save users gas when these structs are saved in storage.

## FighterPhysicalAttributes
Currently when this struct is created for a fighter in `FighterFarm::_createNewFighter` it requires storing data a separate storage slot for each of the 6 fields of the array. However when the values of this struct struct are set in `AiArenaHelper::createPhysicalAttributes` the maximum a value will be set to is 99. Therefore a uint8 would be enough to store each fo these values meaning only one storage slot would be required.

## Fighter
The fighter struct can similarly be optimised for large gas savings.
As this struct includes the above `FighterPhysicalAttributes` struct, those changes will be part of the changes here too.
As for the other fields of the array:
`weight` has a range between 65-95, so this can safely be set as a uint8.
`element` is initialised to 3 for generation 1 so it seems probably that this number won't inflate past 255 in future rounds, so can also be set as a uint8. `id` is the token id of the fighter, it seems reasonable that this number will safely never supass the maximum uint16 (65,535).

The fields of the struct can then be reordered to ensure the minimum storage slots are used.
```solidity
    struct Fighter {
        uint8 weight;
        uint8 element;
        uint16 id;
        uint8 generation;
        uint8 iconsType;
        bool dendroidBool;
        FighterPhysicalAttributes physicalAttributes;
        string modelHash;
        string modelType;
    }
```

**Recommendation**

This results in the following gas savings when minting fighters:
```diff
    | Function Name      | min    | avg     | median  | max    | # calls |
-   | claimFighters      | 16108  | 350552  | 517775  | 517775 | 3       |
+   | claimFighters      | 16096  | 274958  | 404390  | 404390 | 3       |
-   | redeemMintPass     | 327014 | 349995  | 327014  | 395957 | 3       |
+   | redeemMintPass     | 202665 | 221084  | 202665  | 257922 | 3       |
-   | mintFromMergingPool| 364235 | 406101  | 410869  | 432889 | 73      |
+   | mintFromMergingPool| 227724 | 278037  | 297526  | 299646 | 73      |
```

## [G-02] `RankedBattle::claimFighters` could use a struct for `nftsClaimed` to reduce the number of SSTORE's required

Currently it requires two SSTOREs to update the number of fighters & dendroids minted by a user.
However it would be possible to reduce this to one by creating a `FightersMinted` struct.

See the following snippet from the original function:
```solidity
        nftsClaimed[msg.sender][0] += numToMint[0];
        nftsClaimed[msg.sender][1] += numToMint[1];
```

**Recommendation**
First we create the following struct (given the `numToMint` argument passed to `claimFighters` is a uint8 array we'll use uint8s but it can be increased in size to a maximum of uint128 if there's risks uint8 could eventually overflow):
```solidity
    struct FightersMinted {
        uint8 championsMinted;
        uint8 dendroidsMinted;
    }
```
Next the mapping `nftsClaimed` is changed as below:
```diff
-   mapping(address => mapping(uint8 => uint8)) public nftsClaimed;
+   mapping(address => FightersMinted) public nftsClaimed;
```

Then the `claimFighters` functions is changed as below:
```diff
    function claimFighters(
        uint8[2] calldata numToMint,
        bytes calldata signature,
        string[] calldata modelHashes,
        string[] calldata modelTypes
    ) 
        external 
    {
+       FightersMinted memory minted = nftsClaimed[msg.sender];
        bytes32 msgHash = bytes32(keccak256(abi.encode(
            msg.sender, 
            numToMint[0], 
            numToMint[1],
-           nftsClaimed[msg.sender][0],
-           nftsClaimed[msg.sender][1]
+           minted.championsMinted,
+           minted.dendroidsMinted
        )));
        require(Verification.verify(msgHash, signature, _delegatedAddress));
        uint16 totalToMint = uint16(numToMint[0] + numToMint[1]);
        require(modelHashes.length == totalToMint && modelTypes.length == totalToMint);
-       nftsClaimed[msg.sender][0] += numToMint[0];
-       nftsClaimed[msg.sender][1] += numToMint[1];
+       minted.championsMinted += numToMint[0];
+       minted.dendroidsMinted += numToMint[1];
        nftsClaimed[msg.sender] = minted;
        for (uint16 i = 0; i < totalToMint; i++) {
            _createNewFighter(
                msg.sender, 
                uint256(keccak256(abi.encode(msg.sender, fighters.length))),
                modelHashes[i], 
                modelTypes[i],
                i < numToMint[0] ? 0 : 1,
                0,
                [uint256(100), uint256(100)]
            );
        }
    }
```

This results in the following gas savings:
```diff
    | Function Name  | min    | avg     | median  | max    | # calls |
-   | claimFighters  | 16108  | 350552  | 517775  | 517775 | 3       |
+   | claimFighters  | 14005  | 348005  | 515005  | 515005 | 3       |
```

## [G-03] `GameItems::_replenishDailyAllowance` could be optimized by reducing it's two separate mappings into one
The following mappings could be converted into a single one using an `AllowanceInfo` struct:
```solidity
    /// @notice Mapping of address to tokenId to get remaining allowance.
    mapping(address => mapping(uint256 => uint256)) public allowanceRemaining;

    /// @notice Mapping of address to tokenId to get replenish timestamp.
    mapping(address => mapping(uint256 => uint256)) public dailyAllowanceReplenishTime;
```

As these two mappings are both related to the same thing, a users allowance for a specific token id. It would make sense to group them in a single mapping like this:
```solidity
    struct AllowanceInfo {
        uint128 allowanceRemaining;
        uint128 allowanceReplenishTime;
    }

    mapping(address => mapping(uint256 => AllowanceInfo)) public allowanceInfo;
``` 

Full changes to GameItems.sol to facilitiate these changes [here](https://gist.github.com/McCoady/3507665b6f86945f96fa7af2ce411c17)

This results in the following gas savings:
```diff
    | Function Name     | min    | avg    | median | max    | # calls |
-   | mint              | 3444   | 88849  | 107483 | 111658 | 18      |
+   | mint              | 3435   | 71703  | 87304  | 89242  | 18      |
```

# [G-04] `Neuron::claim` makes an unnecessary allowance check that can be removed
The function `claim` makes the following check:
`require(allowance(treasuryAddress, msg.sender) >= amount, "ERC20: claim amount exceeds allowance");`

However `claim` calls `ERC20::transferFrom` which in turn calls `ERC20::_spendAllowance` which also makes this check:
`require(currentAllowance >= amount, "ERC20: insufficient allowance");`

Therefore this initial check can be removed for a gas saving:
```diff
    | Function Name | min  | avg   | median | max   | # calls |
-   | claim         | 4872 | 16170 | 16170  | 27468 | 2       |
+   | claim         | 4934 | 16020 | 16020  | 27106 | 2       |
```

# [G-05] Check number of tokens to mint in `FighterFarm::redeemMintPass` doesn't exceed `MAXIMUM_FIGHTERS_ALLOWED` one time rather than once for every `mintpassIdsToBurn`
Currently the function `redeemMintPass` checks that the users address hasn't exceeded `MAXIMUM_FIGHTERS_ALLOWED` once for each `mintpassIdsToBurn` during `_createNewFighter`. As the majority of tokens will be minted from `redeemMintPas` it would be more gas efficient to check that the user balance + amount of mint passes to redeem doesn't exceed 10 once, rather than checking during each iteration.

**Mitigation**
```diff
    function redeemMintPass(
        uint256[] calldata mintpassIdsToBurn,
        uint8[] calldata fighterTypes,
        uint8[] calldata iconsTypes,
        string[] calldata mintPassDnas,
        string[] calldata modelHashes,
        string[] calldata modelTypes
    ) 
        external 
    {
        require(
            mintpassIdsToBurn.length == mintPassDnas.length && 
            mintPassDnas.length == fighterTypes.length && 
            fighterTypes.length == modelHashes.length &&
            modelHashes.length == modelTypes.length
        );
+       require(balanceOf(msg.sender) + mintpassIdsToBurn.length <= MAX_FIGHTERS_ALLOWED, "Maximum fighters exceeded");
        for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
            require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));
            _mintpassInstance.burn(mintpassIdsToBurn[i]);
            _createNewFighter(
                msg.sender, 
                uint256(keccak256(abi.encode(mintPassDnas[i]))), 
                modelHashes[i], 
                modelTypes[i],
                fighterTypes[i],
                iconsTypes[i],
                [uint256(100), uint256(100)]
            );
        }
    }

    function _createNewFighter(
        address to, 
        uint256 dna, 
        string memory modelHash,
        string memory modelType, 
        uint8 fighterType,
        uint8 iconsType,
        uint256[2] memory customAttributes
    ) 
        private 
    {  
-       require(balanceOf(to) < MAX_FIGHTERS_ALLOWED);
        
        // snip
    }

    // Also update other mint functions

    function claimFighters(
        uint8[2] calldata numToMint,
        bytes calldata signature,
        string[] calldata modelHashes,
        string[] calldata modelTypes
    ) 
        external 
    {
        bytes32 msgHash = bytes32(keccak256(abi.encode(
            msg.sender, 
            numToMint[0], 
            numToMint[1],
            nftsClaimed[msg.sender][0],
            nftsClaimed[msg.sender][1]
        )));
        require(Verification.verify(msgHash, signature, _delegatedAddress));
        uint16 totalToMint = uint16(numToMint[0] + numToMint[1]);
        require(modelHashes.length == totalToMint && modelTypes.length == totalToMint);
+       require(balanceOf(msg.sender) + totalToMint <= MAX_FIGHTERS_ALLOWED, "Max fighters exceeded");

    function mintFromMergingPool(
        address to, 
        string calldata modelHash, 
        string calldata modelType, 
        uint256[2] calldata customAttributes
    ) 
        public 
    {
        require(msg.sender == _mergingPoolAddress);
+       require(balanceOf(to) < MAX_FIGHTERS_ALLOWED, "Max fighers exceeded");
        _createNewFighter(
            to, 
            uint256(keccak256(abi.encode(msg.sender, fighters.length))), 
            modelHash, 
            modelType,
            0,
            0,
            customAttributes
        );
    }
```

# [G-06] Remove unnecessary bytes("random") input in `GameItems::mint::_mint` 

See the following line:
`_mint(msg.sender, tokenId, quantity, bytes("random"));`

As the bytes argument given to `_mint` is not used it would better be replaced with an empty string like so:
`_mint(msg.sender, tokenId, quantity, "");`

The gas savings from doing this are shown here:
```diff
    | Function Name | min  | avg   | median | max    | # calls |
-   | mint          | 3444 | 90897 | 109720 | 111680 | 15      |
+   | mint          | 3444 | 90811 | 109696 | 111547 | 15      |
```

# [G-07] Remove unnecessary check in `MergingPool::pickWinner`

In the function `pickWinner` has the following require statement:
`require(!isSelectionComplete[roundId], "Winners are already selected");`
This check would be necessary if `pickWinner` took a `roundId` argument, hwoever the `roundId` in question is always the current round and is incremented only via this function call. Therefore it is impossible for `isSelectionComplete[roundId]` to ever be true and therefore the check can be removed to save gas.

**Mitigation**
```diff
    function pickWinner(uint256[] calldata winners) external {
        require(isAdmin[msg.sender]);
        require(winners.length == winnersPerPeriod, "Incorrect number of winners");
-       require(!isSelectionComplete[roundId], "Winners are already selected");
        uint256 winnersLength = winners.length;
        address[] memory currentWinnerAddresses = new address[](winnersLength);
        for (uint256 i = 0; i < winnersLength; i++) {
            currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);
            totalPoints -= fighterPoints[winners[i]];
            fighterPoints[winners[i]] = 0;
        }
        winnerAddresses[roundId] = currentWinnerAddresses;
        isSelectionComplete[roundId] = true;
        roundId += 1;
    }
```

# [G-08] The StakeAtRisk contract should read `roundId` from RankedBattle rather than having it's own storage variable

Currently `RankedBattle::setNewRound` increments it's own `RankedBattle::roundId` then calls `_stakeAtRiskInstance.setNewRound` which again increments it's own `StakeAtRisk::roundId` variable. It would be better if `RankedBattle` alone was in charge of storing the `roundId` and `StakeAtRisk` reads from `RankedBattle` to ensure that the round id stays in sync. 

```diff
    | Function Name | min    | avg    | median | max    | # calls |
-   | setNewRound   | 2376   | 32870  | 30962  | 82762  | 108     |
+   | setNewRound   | 2376   | 32371  | 31392  | 63292  | 108     |
```

# [G-09] `GameItems::_replenishDailyAllowance` unnecessarily casts value to uint32
The private function `_replenishDailyAllowance` is called during `GameItems::mint` if it is time to reset a users daily allowance of a particular token id.
In this function there is the following line:
`dailyAllowanceReplenishTime[msg.sender][tokenId] = uint32(block.timestamp + 1 days);`
However as can be seen here:
`mapping(address => mapping(uint256 => uint256)) public dailyAllowanceReplenishTime;` The value is being stored as a uint256 anyway so the casting down to uint32 is unnecessary and removing this will save gas as shown here:

```diff
    | Function Name | min    | avg    | median | max    | # calls |
-   | mint          | 3444   | 88849  | 107483 | 111658 | 18      |
+   | mint          | 3444   | 88836  | 107470 | 111643 | 18      |
```

# [G-10] The constructor of `AiArenaHelper` sets `attributeProbabilities` mapping twice
The constructor first calls `addAttributeProbabilities` to initialise the probabilities, but then back in the core logic of the constructor mistakenly sets them again in a for loop.
**Recommendation**
Remove one of the unnecessary setting methods.

Below showing the difference in deployment cost after the change:
```diff
    | Deployment Cost | Deployment Size
-   | 1741279         | 8445
+   | 1661623         | 8308
```






## [G-11] `Neuron::burnFrom` should store result of `allowance` in memory rather than calling it twice
The `burnFrom` calls `allowance` twice when it could call it once and store that variable in memory.

**Optimized Code**
```diff
    function burnFrom(address account, uint256 amount) public virtual {
+       uint256 senderAllowance = allowance(account, msg.sender);
        require(
-           allowance(account, msg.sender) >= amount,
+           senderAllowance >= amount, 
            "ERC20: burn amount exceeds allowance"
        );
-       uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
+       uint256 decreasedAllowance = senderAllowance - amount;
        _burn(account, amount);
        _approve(account, msg.sender, decreasedAllowance);
    }
```
This results in the following gas savings:
```diff
    | Function Name     | min    | avg    | median | max   | # calls |
-   | burnFrom          | 4761   | 4761   | 4761   | 4761  | 1       |
+   | burnFrom          | 4524   | 4524   | 4524   | 4524  | 1       |
```