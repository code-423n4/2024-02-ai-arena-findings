# Table of contents

- [Table of contents](#table-of-contents)
  - [Summary](#summary)
  - [Gas Optimizations](#gas-optimizations)
    - [Gas Optimizations List](#gas-optimizations-list)
  - [\[G-01\] Fixed size arrays are cheaper than dynamic arrays](#g-01-fixed-size-arrays-are-cheaper-than-dynamic-arrays)
  - [\[G-02\] ++i costs less gas compared to i++ or i += 1](#g-02-i-costs-less-gas-compared-to-i-or-i--1)

## Summary

Most of the gas optimization I know about were already found by bots. The one in this report is the one not included in Automated Findings / Publicly Known Issues

## Gas Optimizations

### Gas Optimizations List

| Number | Optimization Details                                | Instances | Gas Savings |
| :----: | :--------------------------------------------------:| :-------: | :---------: |
| [G-01] | Fixed size arrays are cheaper than dynamic arrays   |     2     |    71414    |
| [G-02] | ++i costs less gas compared to i++ or i += 1        |     1     |      5      |

## [G-01] Fixed size arrays are cheaper than dynamic arrays

In `src/AiArenaHelper.sol` there is no function that adds or removes from the dynamic arrays `attributes` and `defaultAttributeDivisor` therefore we can convert it to a fixed size array.

[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L14-L23)

The code can be refactored as shown below:

```diff
diff --git a/src/AiArenaHelper.sol b/src/AiArenaHelper.sol
index b93dde8..5dcce30 100644
--- a/src/AiArenaHelper.sol
+++ b/src/AiArenaHelper.sol
@@ -14,10 +14,10 @@ contract AiArenaHelper {
     //////////////////////////////////////////////////////////////*/
 
     /// @notice List of attributes
-    string[] public attributes = ["head", "eyes", "mouth", "body", "hands", "feet"]; 
+    string[6] public attributes = ["head", "eyes", "mouth", "body", "hands", "feet"];
 
     /// @notice Default DNA divisors for attributes
-    uint8[] public defaultAttributeDivisor = [2, 3, 5, 7, 11, 13];
+    uint8[6] public defaultAttributeDivisor = [2, 3, 5, 7, 11, 13];
 
     /// The address that has owner privileges (initially the contract deployer).
     address _ownerAddress;
```

**Gas Report before optimization:**

| src/AiArenaHelper.sol:AiArenaHelper contract |                 |        |        |        |         |
|----------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                              | Deployment Size |        |        |        |         |
| 1741279                                      | 8445            |        |        |        |         |
| Function Name                                | min             | avg    | median | max    | # calls |
| addAttributeDivisor                          | 3612            | 26868  | 26868  | 50124  | 2       |
| addAttributeProbabilities                    | 129284          | 129284 | 129284 | 129284 | 1       |
| attributeToDnaDivisor                        | 921             | 1921   | 1921   | 2921   | 2       |
| createPhysicalAttributes                     | 1043            | 68269  | 83687  | 85463  | 53      |
| deleteAttributeProbabilities                 | 2536            | 34295  | 33895  | 66854  | 4       |
| dnaToIndex                                   | 9038            | 9038   | 9038   | 9038   | 1       |
| getAttributeProbabilities                    | 3698            | 6364   | 7698   | 7698   | 3       |
| transferOwnership                            | 2491            | 4027   | 4027   | 5564   | 2       |

**Gas Report after optimization:**

| src/AiArenaHelper.sol:AiArenaHelper contract |                 |        |        |        |         |
|----------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                              | Deployment Size |        |        |        |         |
| 1669865                                      | 8362            |        |        |        |         |
| Function Name                                | min             | avg    | median | max    | # calls |
| addAttributeDivisor                          | 3612            | 25313  | 25313  | 47015  | 2       |
| addAttributeProbabilities                    | 126275          | 126275 | 126275 | 126275 | 1       |
| attributeToDnaDivisor                        | 921             | 1921   | 1921   | 2921   | 2       |
| createPhysicalAttributes                     | 1043            | 64809  | 79610  | 81386  | 53      |
| deleteAttributeProbabilities                 | 2536            | 33065  | 32665  | 64394  | 4       |
| dnaToIndex                                   | 9038            | 9038   | 9038   | 9038   | 1       |
| getAttributeProbabilities                    | 3698            | 6364   | 7698   | 7698   | 3       |
| transferOwnership                            | 2491            | 4027   | 4027   | 5564   | 2       |

From this report we can see that `71414` of gas is saved on deployment.

## [G-02] ++i costs less gas compared to i++ or i += 1

Pre-increments and pre-decrements are cheaper.

`i += 1` and `i -= 1` are the most expensive form.

`i++` costs 6 gas less that `i += 1` while `i--` costs 11 gas less that `i -= 1`.

`++i` costs 5 gas less that `i++` while `--i` costs 5 gas less that `i--`.

[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AAMintPass.sol#L162-L168)

The code can be refactored as shown below:

```diff
diff --git a/src/AAMintPass.sol b/src/AAMintPass.sol
index d0b62ad..89f0421 100644
--- a/src/AAMintPass.sol
+++ b/src/AAMintPass.sol
@@ -162,7 +162,7 @@ contract AAMintPass is ERC721, ERC721Burnable {
     /// @param _receiver The address receiving the mintpass
     /// @param _tokenURI The tokenURI to attach to the mintpass
     function createMintPass(address _receiver, string calldata _tokenURI) private {
-        numTokensOutstanding++;
+        ++numTokensOutstanding;
         uint256 tokenId = numTokensOutstanding + numTokensBurned;
         tokenURIs[tokenId] = _tokenURI;
         _safeMint(_receiver, tokenId);
```

**Gas Report before optimization:**

| src/AAMintPass.sol:AAMintPass contract |                 |        |        |        |         |
|----------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                        | Deployment Size |        |        |        |         |
| 1692111                                | 8414            |        |        |        |         |
| Function Name                          | min             | avg    | median | max    | # calls |
| addAdmin                               | 2506            | 16647  | 22718  | 24718  | 3       |
| approve                                | 25192           | 25192  | 25192  | 25192  | 1       |
| balanceOf                              | 668             | 668    | 668    | 668    | 3       |
| burn                                   | 24835           | 25030  | 25030  | 25225  | 2       |
| claimMintPass                          | 2882            | 136164 | 201187 | 204425 | 3       |
| delegatedAddress                       | 447             | 447    | 447    | 447    | 2       |
| fighterFarmContractAddress             | 383             | 383    | 383    | 383    | 1       |
| founderAddress                         | 426             | 1426   | 1426   | 2426   | 2       |
| isAdmin                                | 643             | 1976   | 2643   | 2643   | 6       |
| mintingPaused                          | 399             | 1399   | 1399   | 2399   | 4       |
| numTokensOutstanding                   | 331             | 1331   | 1331   | 2331   | 2       |
| ownerOf                                | 624             | 624    | 624    | 624    | 3       |
| removeAdmin                            | 2550            | 4337   | 4337   | 6125   | 2       |
| setDelegatedAddress                    | 2504            | 3640   | 3640   | 4777   | 2       |
| setFighterFarmAddress                  | 2506            | 22462  | 22679  | 24679  | 84      |
| setPaused                              | 644             | 756    | 644    | 6084   | 85      |
| tokenURI                               | 2168            | 2168   | 2168   | 2168   | 1       |
| totalSupply                            | 327             | 327    | 327    | 327    | 2       |
| transferOwnership                      | 2526            | 14354  | 14354  | 26182  | 2       |

**Gas Report after optimization:**

| src/AAMintPass.sol:AAMintPass contract |                 |        |        |        |         |
|----------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                        | Deployment Size |        |        |        |         |
| 1691711                                | 8412            |        |        |        |         |
| Function Name                          | min             | avg    | median | max    | # calls |
| addAdmin                               | 2506            | 16647  | 22718  | 24718  | 3       |
| approve                                | 25192           | 25192  | 25192  | 25192  | 1       |
| balanceOf                              | 668             | 668    | 668    | 668    | 3       |
| burn                                   | 24835           | 25030  | 25030  | 25225  | 2       |
| claimMintPass                          | 2882            | 136161 | 201182 | 204420 | 3       |
| delegatedAddress                       | 447             | 447    | 447    | 447    | 2       |
| fighterFarmContractAddress             | 383             | 383    | 383    | 383    | 1       |
| founderAddress                         | 426             | 1426   | 1426   | 2426   | 2       |
| isAdmin                                | 643             | 1976   | 2643   | 2643   | 6       |
| mintingPaused                          | 399             | 1399   | 1399   | 2399   | 4       |
| numTokensOutstanding                   | 331             | 1331   | 1331   | 2331   | 2       |
| ownerOf                                | 624             | 624    | 624    | 624    | 3       |
| removeAdmin                            | 2550            | 4337   | 4337   | 6125   | 2       |
| setDelegatedAddress                    | 2504            | 3640   | 3640   | 4777   | 2       |
| setFighterFarmAddress                  | 2506            | 22462  | 22679  | 24679  | 84      |
| setPaused                              | 644             | 756    | 644    | 6084   | 85      |
| tokenURI                               | 2168            | 2168   | 2168   | 2168   | 1       |
| totalSupply                            | 327             | 327    | 327    | 327    | 2       |
| transferOwnership                      | 2526            | 14354  | 14354  | 26182  | 2       |

From this report we can see that `5` gas was saved in the `claimMintPass` function and `400` of gas is saved on deployment.
