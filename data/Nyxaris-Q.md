## Title: 
Lack of Event Emission on Ownership Transfer

## Description:

 The [`transferOwnership`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L61) function in the `AiArenaHelper` contract allows the current owner to transfer ownership to a new address. However, the function does not emit an event when ownership is transferred. In Solidity, events are crucial for tracking changes to the contract state, especially for off-chain monitoring and indexing services. The absence of an event for ownership transfer makes it difficult to observe and verify changes in ownership through external tools and user interfaces.

## Recommendation: 
It is recommended to define an event, such as [`OwnershipTransferred`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L61), and emit it within the transferOwnership function to log the change of ownership. The event should include the address of the previous owner and the address of the new owner. This will enhance transparency and allow users and external applications to react to changes in contract ownership.

```diff
+  event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

function transferOwnership(address newOwnerAddress) external {
    require(msg.sender == _ownerAddress, "Only the owner can transfer ownership");
+    emit OwnershipTransferred(_ownerAddress, newOwnerAddress);
    _ownerAddress = newOwnerAddress;
}


```