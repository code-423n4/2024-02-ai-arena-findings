## Summary

When calling `_mint(msg.sender, tokenId, quantity, bytes("random"));` in https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L173, passing `bytes("random")` instead of simply `"""` is useless and not elegant.

Also, some gas can be saved.

## Proposed improvements

In https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L173, put:

    _mint(msg.sender, tokenId, quantity, "");

instead of:

    _mint(msg.sender, tokenId, quantity, bytes("random"));

## Gas saving

### Before improvement:
| src/GameItems.sol:GameItems contract |                 |        |        |        |         |
|--------------------------------------|-----------------|--------|--------|--------|---------|
| Function Name                        | min             | avg    | median | max    | # calls |
| mint                                 | 3444            | 101755 | 129620 | 131580 | 11      |

### After improvement:

| src/GameItems.sol:GameItems contract |                 |        |        |        |         |
|--------------------------------------|-----------------|--------|--------|--------|---------|
| Function Name                        | min             | avg    | median | max    | # calls |
| mint                                 | 3444            | 101656 | 129596 | 131447 | 11      |
