
### [G-1] In `GameItems` the variables `name` and `symbol` should be set to constants in order to save gas

**Description:** In the `GameItems` contract the `name` and `symbol` variables are initialized at the start and never changed. Make them constant in order to save gas.

**Recommended Mitigation:**

```diff
    /// @notice The name of this smart contract.
-   string public name = "AI Arena Game Items";

    /// @notice The symbol for this smart contract.
-   string public symbol = "AGI";

    /// @notice The name of this smart contract.
+   string public constant name = "AI Arena Game Items";

    /// @notice The symbol for this smart contract.
+   string public constant symbol = "AGI";
```

### [G-2] In `GameItems` save the string return in `contractURI` to a constant variable

**Description:** In the `GameItems` contract the function `contractURI` returns a string. The more gas-efficient way would be to store that string as a constant variable and then return it.

**Proof of Concept:** Paste this code in Remix.

```javascript
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

contract TestConstantGas {
string private constant URI = "ipfs://bafybeih3witscmml3padf4qxbea5jh4rl2xp67aydqvqsxmyuzipwtpnii";

    function getConstantPubilc() public pure returns(string memory) {
        return URI;
    }

    function returnString() public pure returns(string memory) {
        return "ipfs://bafybeih3witscmml3padf4qxbea5jh4rl2xp67aydqvqsxmyuzipwtpnii";
    }

}
```

**Recommended Mitigation:** Make a constant storage variable to store the string

```diff
+ string public constant URI = "ipfs://bafybeih3witscmml3padf4qxbea5jh4rl2xp67aydqvqsxmyuzipwtpnii"

function contractURI() public pure returns (string memory) {
-       return "ipfs://bafybeih3witscmml3padf4qxbea5jh4rl2xp67aydqvqsxmyuzipwtpnii";
+       return URI;
    }
```

### [G-3] In `GameItems` the `transferOwnership`, `adjustAdminAccess`, `adjustTransferability` and `instantiateNeuronContract` should use onlyOwner modifier to save gas

**Description:** In `GameItems` the `transferOwnership`, `adjustAdminAccess`, `adjustTransferability` and `instantiateNeuronContract` functions all share a common require statement. Make a modifier with the require in order to save gas.

**Recommended Mitigation:** Add an onlyOwner modifier to all the functions mentioned above and remove the require statement in the function.

```diff
+   modifier onlyOwner() {
+       require(msg.sender == _ownerAddress);
+       _;
+   }

+ function transferOwnership(address newOwnerAddress) external onlyOwner {
-        require(msg.sender == _ownerAddress);

+ function adjustAdminAccess(address adminAddress, bool access) external onlyOwner {
-        require(msg.sender == _ownerAddress);

+ function adjustTransferability(uint256 tokenId, bool transferable) external onlyOwner {
-        require(msg.sender == _ownerAddress);

+ function instantiateNeuronContract(address nrnAddress) external onlyOwner {
-        require(msg.sender == _ownerAddress);
```

### [G-4] In `GameItems` the `treasuryAddress` variable should be immutable to save gas

**Description:** In `GameItems` the `treasuryAddress` is initialized in the constructor and never changed. Make it immutable in order to save gas.

**Recommended Mitigation:**

```diff
    /// @notice The address that recieves funds of purchased game items.
-   address public treasuryAddress;
+   address public immutable treasuryAddress;
```

### [G-5] In `Neuron` the `transferOwnership`, `addMinter`, `addStaker`, `addSpender` and `adjustAdminAccess` should use onlyOwner modifier to save gas

**Description:** In `Neuron` the `transferOwnership`, `addMinter`, `addStaker`, `addSpender` and `adjustAdminAccess` functions all share a common require statement. Make a modifier with the require in order to save gas.

**Recommended Mitigation:** Add an onlyOwner modifier to all the functions mentioned above and remove the require statement in the function.

```diff
+   modifier onlyOwner() {
+       require(msg.sender == _ownerAddress);
+       _;
+   }

+ function transferOwnership(address newOwnerAddress) external onlyOwner {
-        require(msg.sender == _ownerAddress);

+ function addMinter(address newMinterAddress) external onlyOwner {
-        require(msg.sender == _ownerAddress);

+ function addStaker(address newStakerAddress) external onlyOwner {
-        require(msg.sender == _ownerAddress);

+ function addSpender(address newSpenderAddress) external onlyOwner {
-        require(msg.sender == _ownerAddress);

+ function adjustAdminAccess(address adminAddress, bool access) external onlyOwner {
-        require(msg.sender == _ownerAddress);
```

### [G-6] In `Neuron` the `setupAirdrop` use uint256 instead of uint32 in the for loop to save gas

**Description:** In `Neuron` the `setupAirdrop` uses a uint32 for the for loop. It would be safer and more gas efficient to use uint256 instead.

**Proof of Concept:** When tested uint256 uses significantly less gas than uint32.

Paste this in Remix

```javascript
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

contract TestUintForLoop {
    uint256[] addresses;

    function uint32Test() external {
        for (uint32 i = 0; i < 1000; i++) {
            addresses.push(i);
        }
    }

    function uint256Test() external {
        for (uint256 i = 0; i < 1000; i++) {
            addresses.push(i);
        }
    }
}
```

**Recommended Mitigation:** Replace uint32 with uint256

```diff
- for (uint32 i = 0; i < recipientsLength; i++) {
+ for (uint256 i = 0; i < recipientsLength; i++) {
```

### [G-7] In `VoltageManager` the `transferOwnership` and `adjustAdminAccess` should use onlyOwner modifier to save gas

**Description:** In `VoltageManager` the `transferOwnership` and `adjustAdminAccess` functions share a common require statement. Make a modifier with the require in order to save gas.

**Recommended Mitigation:** Add an onlyOwner modifier to the functions mentioned above and remove the require statement in the function.


```diff
+   modifier onlyOwner() {
+       require(msg.sender == _ownerAddress);
+       _;
+   }

+ function transferOwnership(address newOwnerAddress) external onlyOwner {
-        require(msg.sender == _ownerAddress);

+ function adjustAdminAccess(address adminAddress, bool access) external onlyOwner {
-        require(msg.sender == _ownerAddress);
```
