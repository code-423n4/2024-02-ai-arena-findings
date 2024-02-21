# NRN total supply can never equal the max

# Relevant GitHub Links:

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L155
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L43


# Vulnerability details

The Neuron token is used for various functions within the platform, including staking, governance, and rewards. The contract sets a maximum supply of NRN tokens during contract creation.

```solidity
/// @notice Maximum supply of NRN tokens.
uint256 public constant MAX_SUPPLY = 10**18 * 10**9;
```

This cap is enforced within the [mint](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L155)  function to prevent the issuance of tokens beyond the predetermined maximum supply. However, the validation logic used to enforce this cap requiring the sum of the current total supply and the amount to be minted to be less than the maximum supply, effectively prohibits the total supply from ever achieving the maximum threshold.

```solidity
function mint(address to, uint256 amount) public virtual {
        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }
```

## **Impact:**

This discrepancy means the platform cannot fully utilize the token supply as intended, creating a deviation from the protocol documentation.

## **Tools Used:**

- Manual analysis

## **Recommendation:**

Adjust the mint function to allow minting up to the maximum supply by changing the condition to totalSupply() + amount <= MAX_SUPPLY.

```solidity
function mint(address to, uint256 amount) public virtual {
        require(totalSupply() + amount <= MAX_SUPPLY, "Trying to mint more than the max supply");
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }
```


## Assessed type

Other

# GameItems contract  breaks ERC1155 specifications

## **Relevant GitHub Links:**

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L194

## **Vulnerability Details:**

The GameItems contract does not fully meet the ERC1155 standard, specifically missing an event emission when updating a token ID's URI. This detail is crucial for marketplaces like OpenSea that rely on such events to track and display updated NFT metadata.

The ERC1155 specification highlights this requirement: [EIP-1155 Specification](https://eips.ethereum.org/EIPS/eip-1155#specification:~:text=%40dev%20MUST%20emit%20when%20the%20URI%20is%20updated%20for%20a%20token%20ID). OpenSea's documentation also mentions the importance of the URI event for metadata updates: [OpenSea Metadata Standards](https://docs.opensea.io/docs/metadata-standards#:~:text=For%20ERC1155%2C%20metadata%20updates%20are%20supported%20via%20the%20specification%20for%20the%20event%20URI%3A).

```solidity
function setTokenURI(uint256 tokenId, string memory _tokenURI) public {
        require(isAdmin[msg.sender]);
        _tokenURIs[tokenId] = _tokenURI;
    }
```

## **Impact:**

This omission will lead to outdated or incorrect item information being shown on NFT marketplaces.

## **Tools Used:**

- Manual analysis

## **Recommendation:**

To align with the ERC1155 standard and ensure that NFT metadata updates are accurately reflected on marketplaces, the setTokenURI function should be updated to emit an event whenever a token's URI is changed:

```solidity
event URI(string value, uint256 indexed id);

function setTokenURI(uint256 tokenId, string memory _tokenURI) public {
    require(isAdmin[msg.sender], "Caller is not an admin");
    _tokenURIs[tokenId] = _tokenURI;
    emit URI(_tokenURI, tokenId);
}
```