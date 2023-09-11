# Repeating the execution of the same delegate should revert instead of triggering the event repeatedly

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

# App should return an order with penalties when its `previewOrder` function is called with unacceptable `minimumReceived` and `maximumSpent` arrays

According to Seaport's documentation, for `previewOrder`, if the parameters do not meet the requirements, it should return an order with penalties rather than directly reverting.

https://github.com/ProjectOpenSea/seaport/blob/main/docs/SeaportDocumentation.md#arguments-and-basic-functionality
> An optimal Seaport app should return an order with penalties when its `previewOrder` function is called with unacceptable `minimumReceived` and `maximumSpent` arrays, so that the caller can learn what the Seaport app expects. 

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L351-L357
```solidity
        if (!(minimumReceived.length == 1 && maximumSpent.length == 1)) revert CreateOffererErrors.NoBatchWrapping();
        if (minimumReceived[0].itemType != ItemType.ERC721 || minimumReceived[0].token != address(this) || minimumReceived[0].amount != 1) {
            revert CreateOffererErrors.MinimumReceivedInvalid(minimumReceived[0]);
        }
        if (maximumSpent[0].itemType != ItemType.ERC721 && maximumSpent[0].itemType != ItemType.ERC20 && maximumSpent[0].itemType != ItemType.ERC1155) {
            revert CreateOffererErrors.MaximumSpentInvalid(maximumSpent[0]);
        }
```

# `transferFrom` in CreateOfferer miss `onlySeaport` modifier

`generateOrder` and `ratifyOrder` both have `onlySeaport` modifier, as one necessary step in seaport's stage, `transferFrom` miss `onlySeaport` modifier.