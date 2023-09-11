## [LOW] Registrator can re-emit Delegate event at any time
In the `DelegateRegistry` contract, all the functions to delegate rights (`delegateAll()`, `delegateContract()`, `delegateERC721()`, `delegateERC20()`, `delegateERC1155()`) can re-emit the event without updating any state variable if they are called several times with the same arguments.

Example with the `DelegateAll` event in `delegateAll()`:
```solidity
File: DelegateRegistry.sol

L44: function delegateAll(address to, bytes32 rights, bool enable) external payable override returns (bytes32 hash) {
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
        emit DelegateAll(msg.sender, to, rights, enable); //@audit can re-emit event even if (enable == true and already enabled) or (enabled == false and from is revoked or empty)
    }
```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L59
If a user calls `delegateAll()` with `enable` set to `true` it will register the hash correctly. If the user calls the same function again with the same arguments it will pass all the `if` statements without triggering any state changes and emit the `DelegateAll` event again.

This can cause issues on the frontend relying on correct event emissions.

## [LOW] Using assembly return can silently end the execution in a multicall
The `checkDelegateForAll()`, `checkDelegateForContract()`, `checkDelegateForERC721()`, `checkDelegateForERC20()` and `checkDelegateForERC1155()` functions use the assembly `return` opcode instead of the regular Solidity `return` to save gas.
However, the assembly `return` opcode is not the same as the regular Solidity `return` statement, it silently ends the execution without reverting the state changes. This can be a problem if any of these functions is called during a `multicall`. It could silently end the execution flow without alerting the user that the execution did not completely go through.

```solidity
File DelegateRegistry.sol

function checkDelegateForAll(address to, address from, bytes32 rights) external view override returns (bool valid) {
        if (!_invalidFrom(from)) {
            valid = _validateFrom(Hashes.allLocation(from, "", to), from);
            if (!Ops.or(rights == "", valid)) valid = _validateFrom(Hashes.allLocation(from, rights, to), from);
        }
        assembly ("memory-safe") {
            // Only first 32 bytes of scratch space is accessed
            mstore(0, iszero(iszero(valid))) // Compiler cleans dirty booleans on the stack to 1, so do the same here
            return(0, 32) // Direct return, skips Solidity's redundant copying to save gas //@audit return will stop execution if called from multicall
        }
    }
```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L173
