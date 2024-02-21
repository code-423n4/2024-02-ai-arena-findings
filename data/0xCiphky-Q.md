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