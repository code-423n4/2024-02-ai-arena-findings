# Summary

Rows https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L151-L157 can be optimized both in readability and in gas consumption.

Considering that:
- if `allGameItemAttributes[tokenId].finiteSupply` is `false`: there's nothing to check
- if `allGameItemAttributes[tokenId].finiteSupply` is `true`: only in this case we must ensure that `quantity <= allGameItemAttributes[tokenId].itemsRemaining`

Then the `if (...)` can be refactored as illustrated below.

# Proposed fix

```diff
diff --git a/src/GameItems.sol b/src/GameItems.sol
--- a/src/GameItems.sol
+++ b/src/GameItems.sol
@@ -148,13 +148,11 @@ contract GameItems is ERC1155 {
         require(tokenId < _itemCount);
         uint256 price = allGameItemAttributes[tokenId].itemPrice * quantity;
         require(_neuronInstance.balanceOf(msg.sender) >= price, "Not enough NRN for purchase");
-        require(
-            allGameItemAttributes[tokenId].finiteSupply == false || 
-            (
-                allGameItemAttributes[tokenId].finiteSupply == true && 
-                quantity <= allGameItemAttributes[tokenId].itemsRemaining
-            )
-        );
+
+        if (allGameItemAttributes[tokenId].finiteSupply == true) {
+            require(quantity <= allGameItemAttributes[tokenId].itemsRemaining);
+        }
+
         require(
             dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp || 
             quantity <= allowanceRemaining[msg.sender][tokenId]
```
