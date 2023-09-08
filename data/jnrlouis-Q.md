## Low Risk Issues

### [L-01] Misleading Events emitted

In `delegateERC721()` from `delegate-registry/src/DelegateRegistry.sol`, Users are allowed to revoke delegations by parsing `enable` as `false`. An `if` statement is used to check if the wallet revoking the delegation is the right wallet, the issue here is that, if it isn't, the `DelegateERC721` event still gets emitted. This can be misleading if these events are read from and used.

This same issue can be found in multiple functions such as the `delegateAll()`, `delegateContract()`, `delegateERC721()`, `delegateERC20()` and `delegateERC1155()`

*There are 5 instances of this issue:*

```javascript
    function delegateERC721(address to, address contract_, uint256 tokenId, bytes32 rights, bool enable) external payable override returns (bytes32 hash) {
        hash = Hashes.erc721Hash(msg.sender, rights, to, tokenId, contract_);
        bytes32 location = Hashes.location(hash);
        address loadedFrom = _loadFrom(location);
        if (enable) {
            if (loadedFrom == Storage.DELEGATION_EMPTY) {
                _pushDelegationHashes(msg.sender, to, hash);
                _writeDelegationAddresses(location, msg.sender, to, contract_);
                _writeDelegation(location, Storage.POSITIONS_TOKEN_ID, tokenId);
                if (rights != "") _writeDelegation(location, Storage.POSITIONS_RIGHTS, rights);
            } else if (loadedFrom == Storage.DELEGATION_REVOKED) {
                _updateFrom(location, msg.sender);
            }
        } else if (loadedFrom == msg.sender) {
            _updateFrom(location, Storage.DELEGATION_REVOKED);
        }
        emit DelegateERC721(msg.sender, to, contract_, tokenId, rights, enable);
    }
```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L82-L99

Recommendation:
Add an else block to revert if `loadedFrom != msg.sender`

```javascript
        } else if (loadedFrom == msg.sender) {
            _updateFrom(location, Storage.DELEGATION_REVOKED);
++      } else {
            revert();
        }
```
