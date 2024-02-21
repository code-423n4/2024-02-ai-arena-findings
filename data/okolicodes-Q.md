## Title
Lack of two-step Process for contract ownership changes

## Impact

Ownable lets you:
transferOwnership from the owner's account to a new one.
If the wrong address is entered while transferring the ownership, the whole protocol will be bricked.

Ownable2Step prevent the contract ownership from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), by requiring that the recipient of the owner permissions actively accept via a contract call of its own.


## Recommended Mitigation Steps
Consider using Openzeppelin Ownable2Step for contract ownership transfers 