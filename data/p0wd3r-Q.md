# Repeating the execution of the same delegate should revert instead of triggering the event repeatedly.

In the `delegate*` function of `DelegateRegistry`, when `enable` is true and `loadedFrom != Storage.DELEGATION_EMPTY != Storage.DELEGATION_REVOKED`, which means that the location already has a delegation, the `DelegateAll` event will be triggered again, causing inconvenience to the listeners and waste user's gas.

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L44-L60
```solidity
    function delegateAll(address to, bytes32 rights, bool enable) external payable override returns (bytes32 hash) {
        hash = Hashes.allHash(msg.sender, rights, to);
        bytes32 location = Hashes.location(hash);
        address loadedFrom = _loadFrom(location);
        if (enable) {
            if (loadedFrom == Storage.DELEGATION_EMPTY) {
                _pushDelegationHashes(msg.sender, to, hash);
                _writeDelegationAddresses(location, msg.sender, to, address(0));
                if (rights != "") _writeDelegation(location, Storage.POSITIONS_RIGHTS, rights);
            } else if (loadedFrom == Storage.DELEGATION_REVOKED) {
                _updateFrom(location, msg.sender);
            }
        } else if (loadedFrom == msg.sender) {
            _updateFrom(location, Storage.DELEGATION_REVOKED);
        }
        emit DelegateAll(msg.sender, to, rights, enable);
    }
```

# In `checkAndPullByType`, there is no authorization check for ERC721 and 1155 tokens like there is for ERC20.

ERC20 checks the allowance before transferring, while ERC721 and 1155 do not check.

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L52-L54
```solidity
        if (IERC20(underlyingContract).allowance(msg.sender, address(this)) < underlyingAmount) {
            revert Errors.InsufficientAllowanceOrInvalidToken();
        }
```

I actually think that this check for ERC20 can be eliminated, because if the allowance is insufficient, it will also revert during the transfer. This check is somewhat redundant and wastes gas.