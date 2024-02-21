|  | issues | instance |
|---|---|---|
| [NC-01]|Weak Signature Verification| 1 |
|[L-01] |Weak Access Control | 2 |


DETAILS 
[NC-01]| Weak Signature Verification: The `claimFighters` function uses the `Verification.verify`() function to verify the signature of the user's claim message. In a situation where the verification logic is flawed, an attacker could potentially forge a signature and claim fighters on behalf of another user.

To Mitigate this risk, Consider using a 
Stronger Signature Verification: Ensuring that the signature verification logic is robust and cannot be bypassed. Example using a well-tested library or implementing additional checks.

|[L-01] |Weak Access Control: In The `claimFighters` there is no check if the caller has the necessary permissions to claim fighters. A malicious attacker can impersonate a legitimate user, and could potentially claim fighters on their behalf.

MITIGATION
 Implement stronger access control measures to ensure that only authorized users can claim fighters. This involves checking the caller's address against a list of authorized addresses.

 In the `adjustTransferability` function, The function takes two arguments: `tokenId` and `transferable`. The tokenId argument specifies the ID of the game item whose transferability is being adjusted, and the transferable argument specifies whether the game item should be transferable or not.
The function first retrieves the `GameItem` struct associated with the specified tokenId from the gameItems mapping. It then checks whether the caller of the function is authorized to adjust the transferability of the game item. This check is performed using the `require` statement, which ensures that the caller is either the owner of the game item, or has been approved to manage all game items on behalf of the owner, or is an operator authorized to manage the game item on behalf of the owner.
If the caller is authorized, the function sets the transferable field of the GameItem struct to the specified value. Finally, the function emits a `TransferabilityChanged` event to notify other contracts or external systems that the transferability of the game item has been changed.
The vulnerability in this function arises from the fact that it allows the transferability of game items to be adjusted by anyone who is authorized to manage the game item, without any further checks or restrictions. This means that a malicious attacker can exploit this function to lock or unlock game items by calling this function with the appropriate `tokenId` and `transferable` values. For example, an attacker could lock a game item that a player is trying to sell, preventing them from making a sale.

Impact: The impact of this exploit can be significant, as it can lead to loss of revenue for players and disruption of the game economy.
Mitigation: Implement stronger access control mechanisms, input validation, Additionally, implementing rate-limiting mechanisms can help prevent attackers from making too many requests in a short period of time.


