## There is duplicate code in the constructor of AiArenaHelper.sol, resulting in unnecessary gas consumption

## POC

The following code exists in the constructor of the AiArenaHelper contract:
```
//AiArenaHelper.sol
constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);// @audit：init

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i]; // Unnecessary gas consumption
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    }
```

The addAttributeProbabilities function is called for initialization, but this sentence duplicates the function of the first sentence in the for loop, resulting in unnecessary gas consumption. The implementation of the addAttributeProbabilities function is as follows:

```
/// @notice Add attribute probabilities for a given generation.
     /// @dev Only the owner can call this function.
     /// @param generation The generation number.
     /// @param probabilities An array of attribute probabilities for the generation.
    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
    }
```

## Recommended Mitigation Steps

```
diff --git a/src/AiArenaHelper.sol b/src/AiArenaHelper.sol
index b93dde8..14527fd 100644
--- a/src/AiArenaHelper.sol
+++ b/src/AiArenaHelper.sol
@@ -42,7 +42,8 @@ contract AiArenaHelper {
         _ownerAddress = msg.sender;
 
         // Initialize the probabilities for each attribute
-        addAttributeProbabilities(0, probabilities);
+        // addAttributeProbabilities(0, probabilities);
+        require(probabilities.length == 6, "Invalid number of attribute arrays");
 
         uint256 attributesLength = attributes.length;
         for (uint8 i = 0; i < attributesLength; i++) {
```

We can add the following code to the test/AiArenaHelper.t.sol file to view the optimized amount of gas:

```
diff --git a/test/AiArenaHelper.t.sol b/test/AiArenaHelper.t.sol
index af3ec28..1510324 100644
--- a/test/AiArenaHelper.t.sol
+++ b/test/AiArenaHelper.t.sol
@@ -190,6 +190,13 @@ contract AiArenaHelperTest is Test {
         assertEq(_helperContract.dnaToIndex(0, 100, "head"), 0);
     }
 
+    function testGasInConstructor() public {
+        uint256 gasBefore = gasleft(); console.log("gasBefore is ",gasBefore);
+        _helperContract = new AiArenaHelper(_probabilities); 
+        uint256 gasAfter = gasleft(); console.log("gasAfter is ",gasAfter);
+        uint256 gasConsumed = gasBefore-gasAfter; console.log("gasConsumed is ",gasConsumed);
+    }
+
     /*//////////////////////////////////////////////////////////////
                                HELPERS
     //////////////////////////////////////////////////////////////*/
```

Before constructor gas optimization, execute `forge test --mt testGasInConstructor -vvv` and get the following results:

```
[⠒] Compiling...
[⠒] Compiling 13 files with 0.8.13
[⠢] Solc 0.8.13 finished in 8.27s
Compiler run successful!

Running 1 test for test/AiArenaHelper.t.sol:AiArenaHelperTest
[PASS] testGasInConstructor() (gas: 1820395)
Logs:
  gasBefore is  9223372036854754470
  gasAfter is  9223372036852935891
  gasConsumed is  1818579

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 6.58ms
 
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

After the constructor gas is optimized, execute `forge test --mt testGasInConstructor -vvv` and get the following results:

```
[⠒] Compiling...
[⠊] Compiling 13 files with 0.8.13
[⠢] Solc 0.8.13 finished in 8.23s
Compiler run successful!

Running 1 test for test/AiArenaHelper.t.sol:AiArenaHelperTest
[PASS] testGasInConstructor() (gas: 1739587)
Logs:
  gasBefore is  9223372036854754470
  gasAfter is  9223372036853016699
  gasConsumed is  1737771

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 6.43ms
 
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

A total of `1818579-1737771` is optimized, which is 80808 gas