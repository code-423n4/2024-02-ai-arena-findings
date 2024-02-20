# Duplicate initialization of attributeProbabilities when initializing AiArenaHelper

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/AiArenaHelper.sol#L45-L49](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/AiArenaHelper.sol#L45-L49)

## Impact

Unnecessary duplicate code exists. More gas is used during deployment.

## Proof of Concept

 `attributeProbabilities` is initialized by `addAttributeProbabilities`, but at below, it does same initialization again.

```solidity
constructor(uint8[][] memory probabilities) {
    _ownerAddress = msg.sender;

    // Initialize the probabilities for each attribute
@>  addAttributeProbabilities(0, probabilities);

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
@>      attributeProbabilities[0][attributes[i]] = probabilities[i];
        attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
    }
}

function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
    require(msg.sender == _ownerAddress);
    require(probabilities.length == 6, "Invalid number of attribute arrays");

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
@>      attributeProbabilities[generation][attributes[i]] = probabilities[i];
    }
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

```diff
constructor(uint8[][] memory probabilities) {
    _ownerAddress = msg.sender;

    // Initialize the probabilities for each attribute
    addAttributeProbabilities(0, probabilities);

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
-       attributeProbabilities[0][attributes[i]] = probabilities[i];
        attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
    }
}
```

# addAttributeDivisor: Need to make sure it's not 0

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/AiArenaHelper.sol#L74](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/AiArenaHelper.sol#L74)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/AiArenaHelper.sol#L107](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/AiArenaHelper.sol#L107)

## Impact

If admin accidentally set it to 0, NFT minting and rerolls will not work.

## Proof of Concept

`attributeToDnaDivisor` must not be set to 0 because it is used for division, so the `addAttributeDivisor` function that sets this value needs to check that the new `attributeDivisors` are non-zero.

## Tools Used

Manual Review

## Recommended Mitigation Steps

```diff
function addAttributeDivisor(uint8[] memory attributeDivisors) external {
    require(msg.sender == _ownerAddress);
    require(attributeDivisors.length == attributes.length);

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
+       require(attributeDivisors[i] > 0);
        attributeToDnaDivisor[attributes[i]] = attributeDivisors[i];
    }
}
```

# tokenURI: should revert when tokenId is not exist

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L402-L404](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L402-L404)

## Impact

Querying a tokenURI for a tokenId that doesn't exist will not revert.

## Proof of Concept

In the `FighterFarm.tokenURI` function, it does not revert even if the `tokenId` has not yet been minted. The [ERC721 specification](https://eips.ethereum.org/EIPS/eip-721#specification) recommends to revert if the tokenId is not exist.

```solidity
/// @title ERC-721 Non-Fungible Token Standard, optional metadata extension
/// @dev See https://eips.ethereum.org/EIPS/eip-721
///  Note: the ERC-165 identifier for this interface is 0x5b5e139f.
interface ERC721Metadata /* is ERC721 */ {
    /// @notice A descriptive name for a collection of NFTs in this contract
    function name() external view returns (string _name);

    /// @notice An abbreviated name for NFTs in this contract
    function symbol() external view returns (string _symbol);

    /// @notice A distinct Uniform Resource Identifier (URI) for a given asset.
@>  /// @dev Throws if `_tokenId` is not a valid NFT. URIs are defined in RFC
    ///  3986. The URI may point to a JSON file that conforms to the "ERC721
    ///  Metadata JSON Schema".
    function tokenURI(uint256 _tokenId) external view returns (string);
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

```diff
function tokenURI(uint256 tokenId) public view override(ERC721) returns (string memory) {
+   require(_exists(tokenId), "ERC721Metadata: URI query for nonexistent token");
    return _tokenURIs[tokenId];
}
```

# viewFighterInfo, `getAllFighterInfo`: generation type mismatch

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterOps.sol#L92](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterOps.sol#L92)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterOps.sol#L43](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterOps.sol#L43)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L433-L436](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L433-L436)

## Impact

`generation` is implicitly type-cast.

## Proof of Concept

The `FighterOps.viewFighterInfo` function returns `generation` as a uint16 type. However, `generation` is a uint8. Therefore, implicit type casting occurs.

```solidity
struct Fighter {
    uint256 weight;
    uint256 element;
    FighterPhysicalAttributes physicalAttributes;
    uint256 id;
    string modelHash;
    string modelType;
@>  uint8 generation;
    uint8 iconsType;
    bool dendroidBool;
}

function viewFighterInfo(
        Fighter storage self, 
        address owner
    ) 
        public 
        view 
        returns (
            address,
            uint256[6] memory,
            uint256,
            uint256,
            string memory,
            string memory,
@>          uint16
        )
    {
        return (
            owner,
            getFighterAttributes(self),
            self.weight,
            self.element,
            self.modelHash,
            self.modelType,
@>          self.generation
        );
    }
```

Also, `FighterFarm.getAllFighterInfo`, which uses `FighterOps.viewFighterInfo` , has the same issue.

```solidity
function getAllFighterInfo(
    uint256 tokenId
)
    public
    view
    returns (
        address,
        uint256[6] memory,
        uint256,
        uint256,
        string memory,
        string memory,
@>      uint16
    )
{
@>  return FighterOps.viewFighterInfo(fighters[tokenId], ownerOf(tokenId));
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Modify `FighterOps.viewFighterInfo`

```diff
function viewFighterInfo(
        Fighter storage self, 
        address owner
    ) 
        public 
        view 
        returns (
            address,
            uint256[6] memory,
            uint256,
            uint256,
            string memory,
            string memory,
-           uint16
+           uint8
        )
    {
        return (
            owner,
            getFighterAttributes(self),
            self.weight,
            self.element,
            self.modelHash,
            self.modelType,
            self.generation
        );
    }
```

And modify `FighterFarm.getAllFighterInfo` 

```diff
function getAllFighterInfo(
    uint256 tokenId
)
    public
    view
    returns (
        address,
        uint256[6] memory,
        uint256,
        uint256,
        string memory,
        string memory,
-       uint16
+       uint8
    )
{
    return FighterOps.viewFighterInfo(fighters[tokenId], ownerOf(tokenId));
}
```

# setupAirdrop: If duplicate addresses exist, only the later amount is valid

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/Neuron.sol#L131-L132](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/Neuron.sol#L131-L132)

## Impact

If there are duplicate addresses in `recipients`, the account will not receive some amount.

## Proof of Concept

If there are duplicate addresses in `recipients`, allowance is overwritten so that only the amounts later in the array are accepted.

```solidity
function setupAirdrop(address[] calldata recipients, uint256[] calldata amounts) external {
    require(isAdmin[msg.sender]);
    require(recipients.length == amounts.length);
    uint256 recipientsLength = recipients.length;
    for (uint32 i = 0; i < recipientsLength; i++) {
@>      _approve(treasuryAddress, recipients[i], amounts[i]);
    }
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Check for address duplicates in `setupAirdrop`, or add the additional amount to the existing allowance.

# claimFighters: User can reuse signatures when you deploy a FighterFarm on other chain

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L199-L206](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L199-L206)

## Impact

If you deploy a FighterFarm on a different chain, user can reuse the signature.

## Proof of Concept

In `claimFighters`, the signature does not include the chainId, so if you deploy a FighterFarm on a different chain, user can reuse the signature.

```solidity
function claimFighters(
    uint8[2] calldata numToMint,
    bytes calldata signature,
    string[] calldata modelHashes,
    string[] calldata modelTypes
) 
    external 
{
@>  bytes32 msgHash = bytes32(keccak256(abi.encode(
        msg.sender, 
        numToMint[0], 
        numToMint[1],
        nftsClaimed[msg.sender][0],
        nftsClaimed[msg.sender][1]
    )));
@>  require(Verification.verify(msgHash, signature, _delegatedAddress));
    ...
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Use ERC712 to prevent signature reuse on other chains.

# burnFrom: Reduces the allowance even when the allowance is max

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/Neuron.sol#L203](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/Neuron.sol#L203)

## Impact

The allowance is no longer type(uint256).max, causing the allowance to be updated every time the user moves the token by `transferFrom`. This consumes additional gas.

## Proof of Concept

In general, if the allowance is set to type(uint256).max, do not modify the allowance. This is the code from Openzeppelin. You can see that it only modifies the allowance when `currentAllowance != type(uint256).max`.

```solidity
function transferFrom(
    address from,
    address to,
    uint256 amount
) public virtual override returns (bool) {
    address spender = _msgSender();
@>  _spendAllowance(from, spender, amount);
    _transfer(from, to, amount);
    return true;
}

function _spendAllowance(
    address owner,
    address spender,
    uint256 amount
) internal virtual {
    uint256 currentAllowance = allowance(owner, spender);
@>  if (currentAllowance != type(uint256).max) {
        require(currentAllowance >= amount, "ERC20: insufficient allowance");
        unchecked {
            _approve(owner, spender, currentAllowance - amount);
        }
    }
}
```

However, the `burnFrom` function modifies the allowance without checking type(uint256).max. Because of this, the allowance is no longer type(uint256).max, and the user has to update the allowance every time the token is moved by `transferFrom`. This consumes additional gas.

```solidity
function burnFrom(address account, uint256 amount) public virtual {
    require(
@>      allowance(account, msg.sender) >= amount, 
        "ERC20: burn amount exceeds allowance"
    );
    uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
    _burn(account, amount);
@>  _approve(account, msg.sender, decreasedAllowance);
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

```diff
function burnFrom(address account, uint256 amount) public virtual {
    require(
        allowance(account, msg.sender) >= amount, 
        "ERC20: burn amount exceeds allowance"
    );
    uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
    _burn(account, amount);
-   _approve(account, msg.sender, decreasedAllowance);
+   _spendAllowance(account, msg.sender, amount);
}
```

# mint: Using unnecessary parameters

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/GameItems.sol#L173](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/GameItems.sol#L173)

## Impact

Using unnecessary parameters.

## Proof of Concept

You don't need to include the `bytes("random")` parameter when calling `_mint`. It's just use more gas and no purpose.

```solidity
function mint(uint256 tokenId, uint256 quantity) external {
        ...
        if (success) {
            ...
@>          _mint(msg.sender, tokenId, quantity, bytes("random"));
            emit BoughtItem(msg.sender, tokenId, quantity);
        }
    }
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

```diff
function mint(uint256 tokenId, uint256 quantity) external {
        ...
        if (success) {
            ...
-           _mint(msg.sender, tokenId, quantity, bytes("random"));
+           _mint(msg.sender, tokenId, quantity, "");
            emit BoughtItem(msg.sender, tokenId, quantity);
        }
    }
```

# Emit ERC4906 event when changing tokenURI

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L389](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L389)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L403](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L403)

## Impact

Changed NFT attributes are not automatically updated on secondary markets.

## Proof of Concept

When minting or rerolling an NFT and its attributes have changed, you can request to update the metadata of the secondary market, such as OpenSea, by generating an event using [ERC4906](https://eips.ethereum.org/EIPS/eip-4906).

[https://docs.opensea.io/docs/metadata-standards#metadata-updates](https://docs.opensea.io/docs/metadata-standards#metadata-updates)

## Tools Used

Manual Review

## Recommended Mitigation Steps

Use ERC4906.

# Use 2 step owner

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L120](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L120)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/AiArenaHelper.sol#L61](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/AiArenaHelper.sol#L61)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/GameItems.sol#L108](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/GameItems.sol#L108)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/MergingPool.sol#L89](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/MergingPool.sol#L89)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/Neuron.sol#L85](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/Neuron.sol#L85)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L167](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L167)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/VoltageManager.sol#L64](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/VoltageManager.sol#L64)

## Impact

There is no way to recover when the owner is set incorrectly by mistake.

## Proof of Concept

In every contract that uses owner authority, the owner changes immediately upon request at `transferOwnership`.

```solidity
function transferOwnership(address newOwnerAddress) external {
    require(msg.sender == _ownerAddress);
    _ownerAddress = newOwnerAddress;
}

```

There is no way to recover if you mistakenly transfer owner authority to the wrong account. If you use a 2-step owner, you can prevent such mistakes because you have to make a contract call with the new owner account and confirm to transfer the owner authority.

## Tools Used

Manual Review

## Recommended Mitigation Steps

Use Ownable2Step.

[https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.9.5/contracts/access/Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.9.5/contracts/access/Ownable2Step.sol)

# The initialization function should only be called once

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L147](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L147)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L155](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L155)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L163](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L163)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/GameItems.sol#L139](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/GameItems.sol#L139)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L201](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L201)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L209](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L209)

## Impact

If the owner's private key is leaked, it's possible to change the address of important contracts, which can cause serious problems.

## Proof of Concept

Initialization functions can be called multiple times. This means that the owner has too much authority. If the owner's private key is leaked, it's possible to change the address of important contracts, which can cause serious problems.

```solidity
function instantiateNeuronContract(address neuronAddress) external {
    require(msg.sender == _ownerAddress);
    _neuronInstance = Neuron(neuronAddress);
}
```

It would be better to prevent variables that do not need to be changed after initialization from being modified by only allowing them to be called once.

## Tools Used

Manual Review

## Recommended Mitigation Steps

Initialize variable once which don’t need to be changed after initialization.

```diff
function instantiateNeuronContract(address neuronAddress) external {
    require(msg.sender == _ownerAddress);
+   require(_neuronInstance == address(0));
    _neuronInstance = Neuron(neuronAddress);
}

```

# AccessControl is being used incorrectly

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/Neuron.sol#L95](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/Neuron.sol#L95)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/Neuron.sol#L103](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/Neuron.sol#L103)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/Neuron.sol#L111](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/Neuron.sol#L111)

## Impact

Using `_setupRole` for the purpose of `grantRole` is deprecated and `_setupRole` should only be used at initialization. Because there is no DEFAULT_ADMIN_ROLE, there is no way to delete the role of the previously registered address.

## Proof of Concept

When using AccessControl, you have to set the DEFAULT_ADMIN_ROLE or each role's admin and use the `grantRole` function to set roles. Using `_setupRole` for the purpose of `grantRole` is deprecated and `_setupRole` should only be used at initialization.

[https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ecd2ca2cd7cac116f7a37d0e474bbb3d7d5e1c4d/contracts/access/AccessControl.sol#L194-L203](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ecd2ca2cd7cac116f7a37d0e474bbb3d7d5e1c4d/contracts/access/AccessControl.sol#L194-L203)

```solidity
/**
     * @dev Grants `role` to `account`.
     *
     * If `account` had not been already granted `role`, emits a {RoleGranted}
     * event. Note that unlike {grantRole}, this function doesn't perform any
     * checks on the calling account.
     *
     * May emit a {RoleGranted} event.
     *
@>   * [WARNING]
     * ====
@>   * This function should only be called from the constructor when setting
     * up the initial roles for the system.
     *
@>   * Using this function in any other way is effectively circumventing the admin
     * system imposed by {AccessControl}.
     * ====
     *
@>   * NOTE: This function is deprecated in favor of {_grantRole}.
     */
```

In addition, because there is no DEFAULT_ADMIN_ROLE, there is no way to delete the role of the previously registered address when it is changed to a new Minter/Staker/Spender in the future.

## Tools Used

Manual Review

## Recommended Mitigation Steps

Set the DEFAULT_ADMIN_ROLE in the constructor. Don't use `_setupRole` in the `addMinter`, `addStaker`, `addSpender` functions, use `grantRole` instead.

# globalStakedAmount is not updated when tokens transfered to the StakeAtRisk contract or from the StakeAtRisk contract

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L496](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L496)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L462](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L462)

## Impact

`globalStakedAmount` has an incorrect value.

## Proof of Concept

When tokens are moved to the StakeAtRisk contract or from the StakeAtRisk contract due to winning or losing in the game, `amountStaked` is updated but `globalStakedAmount` is not updated.

## Tools Used

Manual Review

## Recommended Mitigation Steps

Update `globalStakedAmount` when tokens transfered to the StakeAtRisk contract or from the StakeAtRisk contract.

# There is no NRN deflation logic, so rewards cannot be minted if the NRN token reaches its maximum.

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/Neuron.sol#L156](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/Neuron.sol#L156)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L308](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L308)

## Impact

There is no deflation logic for NRN. If the NRN token reaches its maximum, users can no longer claim game rewards.

## Proof of Concept

NRN tokens can be minted up to 1 billion. At the time of token deployed, 700 million tokens are minted, so an additional 300 million can be minted.

```solidity
uint256 public constant MAX_SUPPLY = 10**18 * 10**9;
uint256 public constant INITIAL_TREASURY_MINT = 10**18 * 10**8 * 2;
uint256 public constant INITIAL_CONTRIBUTOR_MINT = 10**18 * 10**8 * 5;

constructor(address ownerAddress, address treasuryAddress_, address contributorAddress)
    ERC20("Neuron", "NRN")
{
    _ownerAddress = ownerAddress;
    treasuryAddress = treasuryAddress_;
    isAdmin[_ownerAddress] = true;
@>  _mint(treasuryAddress, INITIAL_TREASURY_MINT);
@>  _mint(contributorAddress, INITIAL_CONTRIBUTOR_MINT);
}

function mint(address to, uint256 amount) public virtual {
@>  require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
    require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
    _mint(to, amount);
}
```

Rest of tokens are minted as game rewards, and the number of rewards per round can be changed.

```solidity
function claimNRN() external {
    require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
    uint256 claimableNRN = 0;
    uint256 nrnDistribution;
    uint32 lowerBound = numRoundsClaimed[msg.sender];
    for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
@>      nrnDistribution = getNrnDistribution(currentRound);
@>      claimableNRN += (
            accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
        ) / totalAccumulatedPoints[currentRound];
        numRoundsClaimed[msg.sender] += 1;
    }
    if (claimableNRN > 0) {
        amountClaimed[msg.sender] += claimableNRN;
@>      _neuronInstance.mint(msg.sender, claimableNRN);
        emit Claimed(msg.sender, claimableNRN);
    }
}

function setRankedNrnDistribution(uint256 newDistribution) external {
    require(isAdmin[msg.sender]);
@>  rankedNrnDistribution[roundId] = newDistribution * 10**18;
}

function getNrnDistribution(uint256 roundId_) public view returns(uint256) {
@>  return rankedNrnDistribution[roundId_];
}

```

However, the tokenomics do not include NRN burn or deflation measures. If the NRN token reaches its maximum, users can no longer claim game rewards. Also, if the token supply only increases, the value of the token will gradually decrease. In order for the reward to be valuable, NRN tokens must be regularly burnt.

## Tools Used

Manual Review

## Recommended Mitigation Steps

Add a logic to regularly burn NRN tokens through fees etc.

# Block claimFighters and redeemMintPasses from claiming more than 10

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L495](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L495)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L211](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L211)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L249](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/FighterFarm.sol#L249)

## Impact

If the sum of the user's existing holdings and the number of NFTs being minted exceeds 10, the transaction will revert. 

## Proof of Concept

The recipient can own a maximum of 10 NFTs. If they already own the maximum number, they cannot mint any more.

```solidity
uint8 public constant MAX_FIGHTERS_ALLOWED = 10;

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
@>  require(balanceOf(to) < MAX_FIGHTERS_ALLOWED);
    ...
}
```

Therefore, `claimFighters` or `redeemMintPass` can only mint up to 10 tokens. If the sum of the user's existing holdings and the number of NFTs being minted exceeds 10, the transaction will revert. Therefore, we need to check and display an appropriate error message.

## Tools Used

Manual Review

## Recommended Mitigation Steps

```diff
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
+   require(totalToMint + balanceOf(to) <= MAX_FIGHTERS_ALLOWED, "exceed MAX_FIGHTERS_ALLOWED");
    require(modelHashes.length == totalToMint && modelTypes.length == totalToMint);
    nftsClaimed[msg.sender][0] += numToMint[0];
    nftsClaimed[msg.sender][1] += numToMint[1];
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
+   require(mintpassIdsToBurn.length + balanceOf(msg.sender) <= MAX_FIGHTERS_ALLOWED, "exceed MAX_FIGHTERS_ALLOWED");
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
```

# No need to check isSelectionComplete

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/MergingPool.sol#L121](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/MergingPool.sol#L121)

## Impact

Checks unnecessarily.

## Proof of Concept

`isSelectionComplete` and `roundId` are set atomically and cannot be changed by other functions, so there is no need to check.

```solidity
function pickWinner(uint256[] calldata winners) external {
    require(isAdmin[msg.sender]);
    require(winners.length == winnersPerPeriod, "Incorrect number of winners");
@>  require(!isSelectionComplete[roundId], "Winners are already selected");
    uint256 winnersLength = winners.length;
    address[] memory currentWinnerAddresses = new address[](winnersLength);
    for (uint256 i = 0; i < winnersLength; i++) {
        currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);
        totalPoints -= fighterPoints[winners[i]];
        fighterPoints[winners[i]] = 0;
    }
    winnerAddresses[roundId] = currentWinnerAddresses;
@>  isSelectionComplete[roundId] = true;
@>  roundId += 1;
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

```diff
function pickWinner(uint256[] calldata winners) external {
    require(isAdmin[msg.sender]);
    require(winners.length == winnersPerPeriod, "Incorrect number of winners");
-   require(!isSelectionComplete[roundId], "Winners are already selected");
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

# pickWinner: Same tokenId can be picked multiple times in the same round

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/MergingPool.sol#L118](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/MergingPool.sol#L118)

## Impact

The same tokenId can be picked multiple times in the same round.

## Proof of Concept

The `pickWinner` function does not check for duplicates in the `winners` parameter. Therefore, the same tokenId can be picked multiple times in the same round.

```solidity
function pickWinner(uint256[] calldata winners) external {
    require(isAdmin[msg.sender]);
@>  require(winners.length == winnersPerPeriod, "Incorrect number of winners");
    require(!isSelectionComplete[roundId], "Winners are already selected");
    uint256 winnersLength = winners.length;
    address[] memory currentWinnerAddresses = new address[](winnersLength);
@>  for (uint256 i = 0; i < winnersLength; i++) {
        currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);
@>      totalPoints -= fighterPoints[winners[i]];
@>      fighterPoints[winners[i]] = 0;
    }
    winnerAddresses[roundId] = currentWinnerAddresses;
    isSelectionComplete[roundId] = true;
    roundId += 1;
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Check for duplicates in `winners`.

# Points in the merging pool are not lost when you lose a game, so you can collect points by playing between your own NFTs

## Links to affected code

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/MergingPool.sol#L195](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/MergingPool.sol#L195)

[https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L451](https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L451)

## Impact

You can gather points by playing games with your own NFTs. This can increase the probability of winning rewards.

## Proof of Concept

Game points are counted per round, with the winner gaining and the loser losing points. Therefore, even if you play a game between your own NFTs, one side will suffer a loss, so you cannot gain significant benefits.

However, points accumulated in the Merging pool do not decrease even if you lose the game and are not reset even after the round is over. Therefore, by taking turns playing games between your own NFTs and giving wins and losses to each other, you can continuously increase the Merging pool points. If you create a Poor AI, you can manipulate the wins and losses.

```solidity
function _addResultPoints(
    uint8 battleResult, 
    uint256 tokenId, 
    uint256 eloFactor, 
    uint256 mergingPortion,
    address fighterOwner
) 
    private 
{
    uint256 stakeAtRisk;
    uint256 curStakeAtRisk;
    uint256 points = 0;

    /// Check how many NRNs the fighter has at risk
    stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);

    /// Calculate the staking factor if it has not already been calculated for this round 
    if (_calculatedStakingFactor[tokenId][roundId] == false) {
        stakingFactor[tokenId] = _getStakingFactor(tokenId, stakeAtRisk);
        _calculatedStakingFactor[tokenId][roundId] = true;
    }

    /// Potential amount of NRNs to put at risk or retrieve from the stake-at-risk contract
    curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;
    if (battleResult == 0) {
        /// If the user won the match

        /// If the user has no NRNs at risk, then they can earn points
        if (stakeAtRisk == 0) {
            points = stakingFactor[tokenId] * eloFactor;
        }

        /// Divert a portion of the points to the merging pool
@>      uint256 mergingPoints = (points * mergingPortion) / 100;
        points -= mergingPoints;
@>      _mergingPoolInstance.addPoints(tokenId, mergingPoints);
        ...
    } else if (battleResult == 2) {
        /// If the user lost the match

        /// Do not allow users to lose more NRNs than they have in their staking pool
        if (curStakeAtRisk > amountStaked[tokenId]) {
            curStakeAtRisk = amountStaked[tokenId];
        }
        if (accumulatedPointsPerFighter[tokenId][roundId] > 0) {
            /// If the fighter has a positive point balance for this round, deduct points 
            points = stakingFactor[tokenId] * eloFactor;
            if (points > accumulatedPointsPerFighter[tokenId][roundId]) {
                points = accumulatedPointsPerFighter[tokenId][roundId];
            }
@>          accumulatedPointsPerFighter[tokenId][roundId] -= points;
@>          accumulatedPointsPerAddress[fighterOwner][roundId] -= points;
@>          totalAccumulatedPoints[roundId] -= points;
            if (points > 0) {
                emit PointsChanged(tokenId, points, false);
            }
        } else {
            /// If the fighter does not have any points for this round, NRNs become at risk of being lost
            bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
            if (success) {
                _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
                amountStaked[tokenId] -= curStakeAtRisk;
            }
        }
    }
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Decrease the Merging pool points when losing a game.