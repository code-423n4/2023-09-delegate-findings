
# Low
## [L-01] flahsloan function work with expired token
### Impact
Tokens that have already expired  should be used in the `flashloan` function. An already expired token should only be withdrawn and extended by the principal token owner. Delegate token owners or approved operators should not have the possibility to borrow their underlying tokens for the duration of a single atomic transaction because their rights for the token have expired.

## [L-02] Expired token can be approved and transfer
### Impact 
It is possible to approve a token or use transferFrom to transfer it to another address that has already expired. An expired token should only be withdrawn and extended by the principal token owner

### Recommendation
Add an additional check to determine if the token is expired.

# Non-critical
## [NC-01] Redundant if condition 
### Impact 
The If condiiton 

```solidity
if (underlyingAmount != 0) { //@audit nc: this is useless for erc721
            revert Errors.WrongAmountForType(IDelegateRegistry.DelegationType.ERC721, underlyingAmount);
        }
```

is redundant in the `checkERC721BeforePull` function. In this function, `underlyingAmount` will always be different from 0 and even if it is equal to 0, it will not affect other parts of the codebase.

## [NC-02] Possible approving to zero address and msg.sender
### Impact 
It is possible to approve the `address(0)` or the `msg.sender` in the `setApprovalForAll` function.