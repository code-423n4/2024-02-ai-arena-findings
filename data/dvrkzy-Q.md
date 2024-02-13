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



