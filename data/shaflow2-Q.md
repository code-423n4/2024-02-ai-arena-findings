# Low[1] - "totalBattle" does not accurately represent the number of battles.
## Impact
"totalBattle" represents not the correct total number of battles, but twice the total number of battles.  
## Proof of Concept
When a battle is initiated, the gameServer will invoke the updateBattleRecord function twice: once to update the relevant information of the battle initiator, and once to update the information of the opponent in the battle.  
```solidity
    function updateBattleRecord(
        uint256 tokenId, 
        uint256 mergingPortion,
        uint8 battleResult,
        uint256 eloFactor,
        bool initiatorBool
    ) 
        external 
    {   
        require(msg.sender == _gameServerAddress);
        require(mergingPortion <= 100);
        address fighterOwner = _fighterFarmInstance.ownerOf(tokenId);
        require(
            !initiatorBool ||
            _voltageManagerInstance.ownerVoltageReplenishTime(fighterOwner) <= block.timestamp || 
            _voltageManagerInstance.ownerVoltage(fighterOwner) >= VOLTAGE_COST
        );

        _updateRecord(tokenId, battleResult);
        uint256 stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
        if (amountStaked[tokenId] + stakeAtRisk > 0) {
            _addResultPoints(battleResult, tokenId, eloFactor, mergingPortion, fighterOwner);
        }
        if (initiatorBool) {
            _voltageManagerInstance.spendVoltage(fighterOwner, VOLTAGE_COST);
        }
        totalBattles += 1;
    }
```  
The statement `totalBattles += 1` will be executed twice.  
So, totalBattles will be twice the correct number of battles.

## Tools Used
Manual audit
## Recommended Mitigation Steps
Move the statement `totalBattles += 1`; inside the `if (initiatorBool) { }`block.  
```solidity
        if (initiatorBool) {
            _voltageManagerInstance.spendVoltage(fighterOwner, VOLTAGE_COST);
            totalBattles += 1;
        }
```
  
# Low[2] - The "attributeProbabilities" is initialized twice in the constructor function.
## Impact
In the AiArenaHelper contract, the "attributeProbabilities" is initialized redundantly.  
## Proof of Concept
In the constructor of the AiArenaHelper contract, the function addAttributeProbabilities(0, probabilities) is called, initializing the attributeProbabilities array. However, within the loop, the addAttributeProbabilities array is redundantly initialized again.
Github:[[44](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L44)]
```solidity
   // Initialize the probabilities for each attribute
    addAttributeProbabilities(0, probabilities);

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
        attributeProbabilities[0][attributes[i]] = probabilities[i];
        attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
    }
```
  
```solidity
    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
    }
```
## Tools Used
Manual audit
## Recommended Mitigation Steps
Please choose to remove either addAttributeProbabilities(0, probabilities) or attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];  
I suggest maintaining a consistent style.  
Change addAttributeDivisor to public. Then:

```solidity
    constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;
        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);
        addAttributeDivisor(defaultAttributeDivisor);
    } 
```
or  
```solidity
    constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;
        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 
```

# Low[3] - Consider using a two-step transfer for ownership
## Impact

A copy-paste error or a typo could potentially replace the owner with an address not controlled by the sponsor, resulting in the sponsor losing some control over the contract's permissions.

## Proof of Concept
In the robot competition, a question was raised suggesting a two-step update for the protocol address, but it did not mention updating the owner.  
A copy-paste error or a typo could potentially replace the owner with an address not controlled by the sponsor, resulting in the sponsor losing some control over the contract's permissions.
## Tools Used
Manual audit
## Recommended Mitigation Steps
Consider implementing a two-step procedure for updating protocol addresses, where the recipient is set as pending, and must 'accept' the assignment by making an affirmative call.  



# L[4] - Once set, the allowedBurningAddresses cannot be revoked.
## Impact

The authorized Burning Addresses have significant permissions, enabling them to potentially destroy GameItems from any address. Once an authorized contract is attacked, the destruction cannot be prevented.
## Proof of Concept
The contract can only permit Burning Addresses and cannot be revoked. Once a contract is authorized and subsequently attacked, it cannot be stopped.
GitHub:[[185](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L185)]
```solidity
    /// @notice Sets the allowed burning addresses.
    /// @dev Only the admins are authorized to call this function.
    /// @param newBurningAddress The address to allow for burning.
    function setAllowedBurningAddresses(address newBurningAddress) public {
        require(isAdmin[msg.sender]);
        allowedBurningAddresses[newBurningAddress] = true;
    }

    function burn(address account, uint256 tokenId, uint256 amount) public {
        require(allowedBurningAddresses[msg.sender]);
        _burn(account, tokenId, amount);
    }
```
## Tools Used
Manual audit
## Recommended Mitigation Steps
```solidity
    function setAllowedBurningAddresses(address newBurningAddress,bool state) public {
        require(isAdmin[msg.sender]);
        allowedBurningAddresses[newBurningAddress] = state;
    }
```

# N[1]- Consider implementing more flexible management for GameItems.

Once a GameItem is added, it cannot be deleted, and only the transferability of the game item can be modified by administrators. Other attributes such as daily supply cannot be changed. Please consider flexible management for GameItem and its supply.  
  

For instance, administrators could update the dailyAllowance and itemPrice based on the actual gaming circumstances.

Github:[[208](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L208)]
```solidity
    function createGameItem(
        string memory name_,
        string memory tokenURI,
        bool finiteSupply,
        bool transferable,
        uint256 itemsRemaining,
        uint256 itemPrice,
        uint16 dailyAllowance
    ) 
        public 
    {
        require(isAdmin[msg.sender]);
        allGameItemAttributes.push(
            GameItemAttributes(
                name_,
                finiteSupply,
                transferable,
                itemsRemaining,
                itemPrice,
                dailyAllowance
            )
        );
        if (!transferable) {
          emit Locked(_itemCount);
        }
        setTokenURI(_itemCount, tokenURI);
        _itemCount += 1;
    }
```
# N[2] Potential overflow risk.  
The contract uses totalNumTrained to record the total number of training sessions for all fighters. Its type is uint32.  
Each owner of a fighter can update its model by calling the updateModel function at any time, without any restrictions on the number of updates or time limits.  
If maliciously attacked, there is a risk of overflow for totalNumTrained. In that case, all fighters will be unable to call updateModel to update ModelType and ModelHash.  
```solidity
    /// @notice Aggregate number of training sessions recorded.
    uint32 public totalNumTrained;
```
```solidity
 function updateModel(
        uint256 tokenId,
        string calldata modelHash,
        string calldata modelType
    ) external {
        require(msg.sender == ownerOf(tokenId));
        fighters[tokenId].modelHash = modelHash;
        fighters[tokenId].modelType = modelType;
        numTrained[tokenId] += 1;
        totalNumTrained += 1;
    }
```