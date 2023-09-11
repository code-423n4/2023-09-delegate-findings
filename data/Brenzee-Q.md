## [L-01] `DelegateToken.flashloan` tries to pull assets from `msg.sender` not from where the assets were sent to (`info.receiver`).

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L396
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L402
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L408

In `DelegateToken.flashloan` function assets are not sent to `msg.sender`, but to `info.receiver`. This means that if `msg.sender != info.receiver`, the `info.receiver` has to make a transfer to the `msg.sender`, because `DelegateToken.flashloan` contract expects the asset to be in `msg.sender`.

This makes it inconvenient for the flash loan users since they have to make 1 additional and unnecessary transfer call.

### Recommendation
I suggest modifying all pull functions to allow passing the address, from where the token should be pulled. In this instance that would be `info.receiver`.
In all the other instances where the pull functions are used, this new parameter should be `msg.sender` (like it is now)

## [L-02] `DelegateRegistry.getOutgoingDelegations` and `DelegateRegistry.getOutgoingDelegationHashes` may fail if too many delegations have been made from `DelegateToken`

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L338-L341
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L257-L259
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L267-L269

Whenever a new delegation is made, `_pushDelegationHashes` is called.
```solidity
    function _pushDelegationHashes(address from, address to, bytes32 delegationHash) internal {
        outgoingDelegationHashes[from].push(delegationHash); // @audit If sent by DelegateToken, this will DoS other functions in the future
        incomingDelegationHashes[to].push(delegationHash);
    }
```

This function pushes new delegation hashes to `outgoingDelegationHashes` and `incomingDelegationHashes` arrays. Additionally, there are no functions that remove any of the values from these arrays.

Since `delegateERC20`, `delegateERC721`, and `delegateERC1155` are called by the `DelegateToken` contract, there can be a situation in the future, where `DelegateRegistry.getOutgoingDelegations` and `DelegateRegisry.getOutgoingDelegationHashes` functions will fail because they will run out of gas. (Because `from` is going to be `DelegateToken` address always)

### Recommendation
If both of those functions are to be used in any future deployments, make sure you acknowledge the risks that those functions may run out of gas.

## [L-03] Rebasing tokens will not work with `DelegateToken` contract.

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/DelegateToken.sol#L296-L322
https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/DelegateToken.sol#L353-L386
https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/DelegateToken.sol#L389-L410

`DelegateToken` allows to deposit of any ERC20 tokens. In this current implementation, rebasing tokens will not work with the current system:
- When a user creates a delegation, the specific amount is set.
- After a while when a user withdraws the assets, it will either not send enough tokens or will try to send too many tokens.

### Recommendations
Be aware of rebasing tokens - users may lose the tokens or not receive enough tokens.