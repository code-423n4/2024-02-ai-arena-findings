# Unused event `TokensMinted` in `Neuron` contract

## Relevant GitHub Links

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L20-L21

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L151-L159

## Summary

Event `TokensMinted` is not used but by NatSpec it should be used in `Neuron.mint` function.

## Recommended Mitigation Steps

Add event in `Neuron.mint`.

```diff
  function mint(address to, uint256 amount) public virtual {
      require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
      require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
      _mint(to, amount);
+     emit TokensMinted(to, amount);
  }
```


# No need to inherit `ERC721` by `FighterFarm`

## Relevant GitHub Links

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L16

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L176-L183

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L406-L417

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L443-L452

## Summary

There is no need to inherit `ERC721` by `FighterFarm` because `ERC721Enumerable` already inherits `ERC721`. 

## Recommended Mitigation Steps

1. Remove inheritance of `ERC721` by `FighterFarm`:
```diff
-   contract FighterFarm is ERC721, ERC721Enumerable {
+   contract FighterFarm is ERC721Enumerable {
```

2. Fix override statements in functions `FighterFarm.tokenURI`, `FighterFarm.supportsInterface`:

```diff
-   function tokenURI(uint256 tokenId) public view override(ERC721) returns (string memory) {
+   function tokenURI(uint256 tokenId) public view override returns (string memory) {
        return _tokenURIs[tokenId];
    }
```

```diff
    function supportsInterface(bytes4 _interfaceId)
        public
        view
-       override(ERC721, ERC721Enumerable)
+       override
        returns (bool)
    {
        return super.supportsInterface(_interfaceId);
    }
```

3. Remove redundant empty `FighterFarm._beforeTokenTransfer` function:

```diff
-   /*//////////////////////////////////////////////////////////////
-                           INTERNAL FUNCTIONS
-   //////////////////////////////////////////////////////////////*/
-
-   /// @notice Hook that is called before a token transfer.
-   /// @param from The address transferring the token.
-   /// @param to The address receiving the token.
-   /// @param tokenId The ID of the NFT being transferred.
-   function _beforeTokenTransfer(address from, address to, uint256 tokenId)
-       internal
-       override(ERC721, ERC721Enumerable)
-   {
-       super._beforeTokenTransfer(from, to, tokenId);
-   }
```