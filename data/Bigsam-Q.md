1. # Lines of code

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L68-L76


# Vulnerability details

## Impact
Owner can set over-estimated values of AttributeDivisor. It is considered a best practice to incorporate validation or constraints on AttributeDivisor values to guarantee they fall within a reasonable range. This practice is crucial for preventing unintended behavior and ensuring that assigned AttributeDivisor remain within range, thus mitigating the risk of unexpected issues.


## Recommended Mitigation Steps
introduction of a maxAttributeDivisor value  with a constant representing the maximum allowed value, providing flexibility for adjustment based on specific requirements.

A nested loop to check and ensure that each individual AttributeDivisor values do not surpass the specified maximum value.

2. # Lines of code

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L131-L138


# Vulnerability details

## Impact
Owner can set over-estimated values as attribute probabilities, thereby creating a wide and unreasonable gap between individual fighters. It is considered a best practice to incorporate validation or constraints on probability values to guarantee they fall within a reasonable range(<100). This practice is crucial for preventing unintended behavior and ensuring that assigned probabilities remain meaningful, thus mitigating the risk of unexpected issues.


## Recommended Mitigation Steps
introduction of a maxProbabilityValue  with a constant (Based on the game examples preferably 99) representing the maximum allowed probability value, providing flexibility for adjustment based on specific requirements.

A nested loop to check and ensure that each individual probability values do not surpass the specified maximum value.







