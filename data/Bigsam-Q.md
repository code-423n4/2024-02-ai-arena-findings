1. # Lines of code

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L169-L186


# Vulnerability details

it's a good practice to handle the situation where the loop never breaks the condition to ensure that attributeProbabilityIndex gets a meaningful value.
When such conditions arises attributeProbabilityindex is not set and function createPhysicalAttributes depends on this value

 uint256 attributeIndex = dnaToIndex(generation, rarityRank, attributes[i]);
                    finalAttributeProbabilityIndexes[i] = attributeIndex;

No attributeProbabilityindex , no phsyical attributes can be created.
If cumProb is always less than rarityRank for all iterations of the loop, it means that the loop never encounters a condition where cumProb becomes greater than or equal to rarityRank. In this case, the attributeProbabilityIndex will remain unset or be the default value (zero for an uninitialized uint256 variable).

Solution: a value can be set to handle the situation after the loop.


2. # Lines of code

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L68-L76


# Vulnerability details

## Impact
Owner can set over-estimated values of AttributeDivisor. It is considered a best practice to incorporate validation or constraints on AttributeDivisor values to guarantee they fall within a reasonable range. This practice is crucial for preventing unintended behavior and ensuring that assigned AttributeDivisor remain within range, thus mitigating the risk of unexpected issues.


## Recommended Mitigation Steps
introduction of a maxAttributeDivisor value  with a constant representing the maximum allowed value, providing flexibility for adjustment based on specific requirements.

A nested loop to check and ensure that each individual AttributeDivisor values do not surpass the specified maximum value.

3. # Lines of code

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L131-L138


# Vulnerability details

## Impact
Owner can set over-estimated values as attribute probabilities, thereby creating a wide and unreasonable gap between individual fighters. It is considered a best practice to incorporate validation or constraints on probability values to guarantee they fall within a reasonable range(<100). This practice is crucial for preventing unintended behavior and ensuring that assigned probabilities remain meaningful, thus mitigating the risk of unexpected issues.


## Recommended Mitigation Steps
introduction of a maxProbabilityValue  with a constant (Based on the game examples preferably 99) representing the maximum allowed probability value, providing flexibility for adjustment based on specific requirements.

A nested loop to check and ensure that each individual probability values do not surpass the specified maximum value.







