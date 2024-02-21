*src/Neuron.sol*
# QA-01 User should not approve Spender more than his balance.

## Description
The role for approvingSpender is trusted but then also if there's any mislead happens it can be a loss for the user.
As in the ``function approverSpender()`` the user can approve more than the balance of his account.

## Tools
Manual review

## Mitigation Step
```   
 function approveSpender(address account, uint256 amount) public {
        require(
            hasRole(SPENDER_ROLE, msg.sender),  
            "ERC20: must have spender role to approve spending"
        );
+        require(amount <= balanceOf(account));
        _approve(account, msg.sender, amount);
    }    

```