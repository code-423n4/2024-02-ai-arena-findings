## Winners of later rounds in `MergingPool` contract are charged more gas to claim their winnings

## Explanation

The `MergingPool::claimRewards` function is not gas efficient as each loop through another round requires 2 extra `SHA3` opcodes, 2 extra `SLOAD`s and 1 extra `SSTORE`. This results in approximately 3000 extra gas per round (see PoC below) and is effectively an arbitrary penalty for winners of later rounds.

Since each round is intended to last 1 week, after 2 years there will have been +100 rounds. If a user wins round 99 instead of round 0, they must pay an additional arbitrary penalty of ~300,000 gas, which is inequitable. 

This can be seen in the snippet and full function code below:

```solidity
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            numRoundsClaimed[msg.sender] += 1;
            winnersLength = winnerAddresses[currentRound].length;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L149-L151

```solidity
    function claimRewards(
        string[] calldata modelURIs, 
        string[] calldata modelTypes,
        uint256[2][] calldata customAttributes
    ) 
        external 
    {
        uint256 winnersLength;
        uint32 claimIndex = 0;
        uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            numRoundsClaimed[msg.sender] += 1;
            winnersLength = winnerAddresses[currentRound].length;
            for (uint32 j = 0; j < winnersLength; j++) {
                if (msg.sender == winnerAddresses[currentRound][j]) {
                    _fighterFarmInstance.mintFromMergingPool(
                        msg.sender,
                        modelURIs[claimIndex],
                        modelTypes[claimIndex],
                        customAttributes[claimIndex]
                    );
                    claimIndex += 1;
                }
            }
        }
        if (claimIndex > 0) {
            emit Claimed(msg.sender, claimIndex);
        }
    }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L139-L167

## PoC
Add the following test to the `MergingPoolTest` test suite:

```solidity
    function testDemonstrateGasChange() external {
        _mintFromMergingPool(_ownerAddress);
        _mintFromMergingPool(_DELEGATED_ADDRESS);
        assertEq(_fighterFarmContract.ownerOf(0), _ownerAddress);
        assertEq(_fighterFarmContract.ownerOf(1), _DELEGATED_ADDRESS);
        uint256[] memory _winners = new uint256[](2);
        _winners[0] = 0;
        _winners[1] = 1;

        // Moving to round 99. Comment out if doing round 0
        vm.store(address(_mergingPoolContract), bytes32(uint256(1)), bytes32(uint256(99)));

        // winners of roundId 99 are picked, or otherwise
        _mergingPoolContract.pickWinner(_winners);
        string[] memory _modelURIs = new string[](1);
        _modelURIs[0] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        string[] memory _modelTypes = new string[](1);
        _modelTypes[0] = "original";
        uint256[2][] memory _customAttributes = new uint256[2][](1);
        _customAttributes[0][0] = uint256(1);
        _customAttributes[0][1] = uint256(80);

        // other winner claims rewards for previous roundIds
        vm.prank(_DELEGATED_ADDRESS);
        _mergingPoolContract.claimRewards(_modelURIs, _modelTypes, _customAttributes);
        uint256 numRewards = _mergingPoolContract.getUnclaimedRewards(_DELEGATED_ADDRESS);
        emit log_uint(numRewards);
        assertEq(numRewards, 0);
    }
```

Run:

```shell
forge test --match-test testDemonstrateGasChange -vvv
```

Output for round 99 and round 0 respectively:

```txt
No files changed, compilation skipped

Running 1 test for test/MergingPool.t.sol:MergingPoolTest
[32m[PASS][0m testDemonstrateGasChange() (gas: 1640704)
Logs:
  0

Test result: [32mok[0m. [32m1[0m passed; [31m0[0m failed; [33m0[0m skipped; finished in 7.30ms
 
Ran 1 test suites: [32m1[0m tests passed, [31m0[0m failed, [33m0[0m skipped (1 total tests)
Compiling 1 files with 0.8.13
Solc 0.8.13 finished in 6.44s
Compiler run [32msuccessful![0m

Running 1 test for test/MergingPool.t.sol:MergingPoolTest
[32m[PASS][0m testDemonstrateGasChange() (gas: 1361907)
Logs:
  0

Test result: [32mok[0m. [32m1[0m passed; [31m0[0m failed; [33m0[0m skipped; finished in 6.33ms
 
Ran 1 test suites: [32m1[0m tests passed, [31m0[0m failed, [33m0[0m skipped (1 total tests)
```

As we can see from the output, round 99 winner test cost 1640704 gas, whereas exact same setup except for round 0 cost only 1361907 gas, a 278,797 gas difference. Thus a winner at round 99 had to pay 278,797 more gas than winner 0, which should not occur.

## Recommended Mitigation Steps
Use a storage pointer for `numRoundsClaimed[msg.sender]` to avoid an extra `SHA3` opcode each round, and otherwise refactor the code to avoid unnecessary extra storage reads and writes