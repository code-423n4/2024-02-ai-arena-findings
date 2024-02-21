# Quality Assurance Report

### Low-Severity

## [L-01] NFT should not be minted on 0 tokenId

The current implementation of storing fighter structs in the fighters array by using the tokenId as an index may lead to inefficiencies, especially when minting NFTs with tokenId 0.

```solidity
File : src/FighterFarm.sol

68:    /// @notice List of all fighter structs, accessible by using tokenId as index.
69:    FighterOps.Fighter[] public fighters;

```

[FighterFarm.sol#L68C1-L69C42](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L68C1-L69C42)

## [L-02] Add require check in `incrementGeneration` function to ensure that generation won't exceeds the limit of uint8

The `incrementGeneration` function in your contract appears to be designed to increment the generation count for a specified fighterType. However, there's a consideration to ensure that the generation count doesn't exceed 255.

```solidity
File : src/FighterFarm.sol

42: uint8[2] public generation = [0, 0];

129:     function incrementGeneration(uint8 fighterType) external returns (uint8) {
130:        require(msg.sender == _ownerAddress);
131:        generation[fighterType] += 1;
132:        maxRerollsAllowed[fighterType] += 1;
133:        return generation[fighterType];
134:    }

```

[src/FighterFarm.sol#L129C1-L134C6](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L129C1-L134C6)

## [L-03] Change the data type of `state variable` `totalNumTrained` `uint32` to `uint256`

The maximum value that can be held by a `uint32` variable is 2^32 - 1, which is 4,294,967,295. If someone wants to reach the maximum value of a `uint32` variable by using a loop in a smart contract, it is technically feasible. However, if there's a possibility that the value could exceed 4,294,967,295 it might be prudent to use `uint256` to ensure sufficient range.

```diff
File : src/FighterFarm.sol

44:        /// @notice Aggregate number of training sessions recorded.
-45:       uint32 public totalNumTrained;
+45:       uint256 public totalNumTrained;
...
283:       function updateModel(
284:       uint256 tokenId,
285:       string calldata modelHash,
286:       string calldata modelType
287:       )
288:       external
289:    {
290:        require(msg.sender == ownerOf(tokenId));
291:        fighters[tokenId].modelHash = modelHash;
292:        fighters[tokenId].modelType = modelType;
293:        numTrained[tokenId] += 1;
294:        totalNumTrained += 1;
295:    }

```

[src/FighterFarm.sol#L283C1-L295C6](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L283C1-L295C6)

## [L-04] Asking for user input or a choice before setting the value in the mapping

Asking the user for a choice provides transparency and allows users to have control over certain aspects of the contract. Users may prefer to have a say in whether a particular address is allowed to perform burning operations.

```solidity
File : src/GameItems.sol

185:    function setAllowedBurningAddresses(address newBurningAddress) public {
186:        require(isAdmin[msg.sender]);
187:        allowedBurningAddresses[newBurningAddress] = true;
188:    }

```

[src/GameItems.sol#L185C1-L188C6](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L185C1-L188C6)

```

// Ask for user choice (you might implement this based on your specific use case)
bool userChoice = getUserChoice(); // You need to define and implement the getUserChoice function

// Set the mapping based on user choice
allowedBurningAddresses[newBurningAddress] = userChoice;

```

In this modification, you would need to implement a function (getUserChoice in this example) that interacts with the user or a user interface to obtain their choice (whether to set the value to true or false). This approach provides more flexibility and user control over the setting of burning addresses.

## [L-05] Don't emit default value

`_itemCount` is being updated after emitting the `Locked` event, and it points out that emitting an event with `_itemCount` equal to 0 might not be desired. This is because the default value of `_itemCount` is 0, and emitting an event with a default value might not provide meaningful information. Since 0 is the default value, it cannot be put in the mapping value.

```solidity
File : src/GameItems.sol

208:    function createGameItem(
...
230:    if (!transferable) {
231:          emit Locked(_itemCount);
232:        }
233:        setTokenURI(_itemCount, tokenURI);
234:        _itemCount += 1;
235:    }

```

[src/GameItems.sol#L230C9-L235C6](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L230C9-L235C6)

## [L-06] Use `msg.sender` instead of explicitly passing the address of the `owner` when `deploying` the contract

You can set the \_ownerAddress directly in the `constructor` using `msg.sender`.

```diff
File : src/Neuron.sol

64:   /// @param ownerAddress The address of the owner who deploys the contract
...
-68:    constructor(address ownerAddress, address treasuryAddress_, address contributorAddress)
+68:    constructor(address treasuryAddress_, address contributorAddress)
69:        ERC20("Neuron", "NRN")
70:    {
-71:       _ownerAddress = ownerAddress;
+71:       _ownerAddress = msg.sender;
72:        treasuryAddress = treasuryAddress_;
73:        isAdmin[_ownerAddress] = true;
74:        _mint(treasuryAddress, INITIAL_TREASURY_MINT);
75:        _mint(contributorAddress, INITIAL_CONTRIBUTOR_MINT);
76:    }

```

[src/Neuron.sol#L68C1-L76C6](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L68C1-L76C6)

## [L-07] Use `<=` for `max supply` instead of `<`

`MAX_SUPPLY` should be allowed, otherwise the maximum supply won't be minted.

```diff
File : src/Neuron.sol

155:    function mint(address to, uint256 amount) public virtual {
-156:       require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
+157:       require(totalSupply() + amount <= MAX_SUPPLY, "Trying to mint more than or equal to the max supply");
158:        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
159:        _mint(to, amount);
160:    }

```

[src/Neuron.sol#L155C1-L159C6](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L155C1-L159C6)

## [L-08] Do not use the method of removing stakers, instead use status in place of true

Using a `status` method instead of a simple boolean value like "true" or "false" can offer additional flexibility and provide more information about the state or condition being represented.

```solidity
File : src/FighterFarm.sol

139:    function addStaker(address newStaker) external {
140:        require(msg.sender == _ownerAddress);
141:        hasStakerRole[newStaker] = true;
142:    }

```

[src/FighterFarm.sol#L139C1-L142C6](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L139C1-L142C6)