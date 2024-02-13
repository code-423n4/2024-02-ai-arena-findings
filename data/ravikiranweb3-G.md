## GameItems::mint()
User can mint upto a max of daily allowance for any game item. As there is no accumulation of allowances across days, the daily allowance is the max limit for minting.

Even when the replenishDailyAllowance() is due, it will set daily limit only.
Hence, the **quantity to mint should be less than or equal to daily allowance**.
The check for quantity is done late in the flow of mint() function.

Doing the check early enough will save lot of gas.

```
function mint(uint256 tokenId, uint256 quantity) external {
        require(tokenId < _itemCount);
===>    require(allGameItemAttributes[tokenId].dailyAllowance >= quantity, "Invaid quantity");
```