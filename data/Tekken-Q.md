## Low
### [L-1] [`MergingPool::getFighterPoints`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L205) should take two arguments instead of one, to be able to preview a range of fighter's points, instead of iterating through all of them up to the `maxId`

**Impact:** The function `getFighterPoints` currently does not provide a way to set a starting index for points to be retrieved. It always starts from the beginning (i = 0). If you want to provide a way to get fighter points inside a range, you should include a startId parameter to the function.

**Recommended Adjustment:** 
```js
    function getFighterPoints(uint256 startId, uint256 endId) public view returns(uint256[] memory) {
        uint256 range = endId - startId;
        uint256[] memory points = new uint256[](range);
        for (uint256 i = startId; i < range; i++) {
            points[i] = fighterPoints[i];
        }
        return points;
    }
}
```

### [L-2] Unused [event `Neuron::TokensMinted`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L21) which is missing from the [`Neuron::constructor`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L74-L75) and [`Neuron::mint`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L158), to emit of the event of minting tokens

**Description:** Event `TokensMinted` is not emitted during minting in the `constructor` and `mint` functions. The ERC20's event for the `_mint` function only displays a Transfer function, which could be cofusing when inspecting the events, even-though it is a transfer, it's still a minting function and that should be made clear by utilizing the unused `TokensMinted` event. 

**Impact:** It's bad practice to miss out on emitting events for essential state modifications or significant function calls.

**Recommended Adjustment:**
```diff
    constructor(address ownerAddress, address treasuryAddress_, address contributorAddress)
        ERC20("Neuron", "NRN")
    {
        _ownerAddress = ownerAddress;
        treasuryAddress = treasuryAddress_;
        isAdmin[_ownerAddress] = true;
        _mint(treasuryAddress, INITIAL_TREASURY_MINT);
        _mint(contributorAddress, INITIAL_CONTRIBUTOR_MINT);
+       emit TokensMinted(treasuryAddress, INITIAL_TREASURY_MINT)
+       emit TokensMinted(contributorAddress, INITIAL_CONTRIBUTOR_MINT)
    }
    .
    .
    .
    function mint(address to, uint256 amount) public virtual {
        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
+       emit TokensMinted(to, amount)
    }
``` 

## Informational
### [I-1] In the function `Neuron::claim`, [`transferFrom`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L143) is not checked for success

**Recommended Adjustment:** 
```diff
    function claim(uint256 amount) external {
        require(
            allowance(treasuryAddress, msg.sender) >= amount, 
            "ERC20: claim amount exceeds allowance"
        );
-       transferFrom(treasuryAddress, msg.sender, amount);
-       emit TokensClaimed(msg.sender, amount);
+       bool success = transferFrom(treasuryAddress, msg.sender, amount);
+       if (success) {
+           emit TokensClaimed(msg.sender, amount);
+       }
    }
```
