# Consider adding validation for the `quantity` parameter in `GameItems::mint()` to avoid a panic 0x11 if the quantity is greater than the remaining allowance.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L147

## Description

This case only happens when the first statement of the folllowing require is evaluated to true, thus the second condition won't be checked:

```js
require(
    dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp || 
    quantity <= allowanceRemaining[msg.sender][tokenId]
);
```

The function, fortunately, will revert anyway because of this line:

```js
allowanceRemaining[msg.sender][tokenId] -= quantity;
```

But it will throw a panic `0x11 arithmetic error` doing so.

The [Solidity documentation](https://docs.soliditylang.org/en/v0.8.23/control-structures.html#panic-via-assert-and-error-via-require) states the following:

`Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix.`

## Impact

It do

## Proof of Concept

Add the following unit test to `test/GameItems.t.sol`:

```js
function testMintPanic() public {
    _fundUserWith4kNeuronByTreasury(address(100));
    vm.startPrank(address(100));
    _gameItemsContract.mint(0, 10);
    vm.warp(block.timestamp + 1 days + 1 seconds);
    _gameItemsContract.mint(0, 12);
    vm.stopPrank();
}
```

## Reccomended Mitigation

Switch the second condition of the require statement pointed above at the first lines of the function so that it will be always checked.

```js
function mint(uint256 tokenId, uint256 quantity) external {
    require(tokenId < _itemCount);
    require(quantity <= allowanceRemaining[msg.sender][tokenId], "Quantity greater than allowance");
    // ...
}
```