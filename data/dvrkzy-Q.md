# Use a two-step ownership transfer approach

Summary
===============
The owner can unintentionally transfer the ownership to the wrong addres using the `transferOwnership` function

Impact
===============
This could brick a lot of the functionality of the contracts as most functions require `msg.sender == _ownerAddress`

Mitigation
===============
Make sure to use a two-step ownership transfer approach by using `Ownable2Step` from OpenZeppelin

Code Snippet
===============

    function transferOwnership(address newOwnerAddress) external {
            require(msg.sender == _ownerAddress);
            _ownerAddress = newOwnerAddress;
    }



`isAdmin` mapping not updated during ownership transfer
===============
Summary
---------------
The owner can transfer ownership in most contracts using the following function:

    function transferOwnership(address newOwnerAddress) external {
            require(msg.sender == _ownerAddress);
            _ownerAddress = newOwnerAddress;
        }
However this function doesn't update the `isAdmin` mapping.

Impact
---------------
After the transfer the old owner will still have some power because the `isAdmin` mapping is not updated to `false` for his address during ownership transfer. Instead it has to be updated manually by the new owner using `adjustAdminAccess()`

This will allow the old owner to call some functions like `adjustAllowedVoltageSpenders()` in `VoltageManager.sol` which `require(isAdmin[msg.sender]);`

Moreover `isAdmin` is also not updated to `true` for the new owner during ownership transfer. He again has to manually change it for himself using `adjustAdminAccess()` or he won't be able to call `adjustAllowedVoltageSpenders()`

Mitigation
---------------
Use the following code:

    function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
        isAdmin[msg.sender] = false;
        isAdmin[_ownerAddress] = true;
    }



