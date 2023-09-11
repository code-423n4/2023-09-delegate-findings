## Finding Summary 

| ID | Description | Severity |
| - | - | :-: |
| [L-01](#l-01-the-owner-of-PT-can-call-extend-even-after-expiry) | The owner of PT can call `extend()` even after expiry | Low |

## [L-01] The owner of PT can call `extend()` even after expiry

In the current implementation, `extend()` function is used by the PT (Principal Token) owner to extend the period of delegation for DT (Delegate Token) owner. The point is that it should not be allowed to extend() the period after expiry as this is what expiry is needed for:

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L325-336
```
function extend(uint256 delegateTokenId, uint256 newExpiry) external {
        StorageHelpers.revertNotMinted(delegateTokenInfo, delegateTokenId);
        Helpers.revertOldExpiry(newExpiry);
        uint256 previousExpiry = StorageHelpers.readExpiry(delegateTokenInfo, delegateTokenId);
        if (newExpiry <= previousExpiry) revert Errors.ExpiryTooSmall();
        if (PrincipalToken(principalToken).isApprovedOrOwner(msg.sender, delegateTokenId)) {
            StorageHelpers.writeExpiry(delegateTokenInfo, delegateTokenId, newExpiry);
            emit ExpiryExtended(delegateTokenId, previousExpiry, newExpiry);
            return;
        }
        revert Errors.NotApproved(msg.sender, delegateTokenId);
    }
```

### Recommendation

Add the check to the `extend()` function so that it'd revert if block.timestamp > expiry
