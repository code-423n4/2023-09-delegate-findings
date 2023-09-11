# [Low] `unchecked` blocks with additions may overflow
There aren't any checks to avoid an overflow which can happen inside an unchecked block, so the following additions may overflow silently.
[calculateIncreasedAmount](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenRegistryHelpers.sol#L136)
```
    function calculateIncreasedAmount(address delegateRegistry, bytes32 registryHash, uint256 increaseAmount) internal view returns (uint256) {
        unchecked {
            return
                uint256(IDelegateRegistry(delegateRegistry).readSlot(bytes32(uint256(RegistryHashes.location(registryHash)) + RegistryStorage.POSITIONS_AMOUNT))) + increaseAmount;
        }
    }
```







# [Info] PT._checkDelegateTokenCaller should be changed to a modifier
This function checks whether the msg.sender is equal to delegateToken, could be written as a modifier.
```
    function _checkDelegateTokenCaller() internal view {
        if (msg.sender == delegateToken) return;
        revert CallerNotDelegateToken();
    }
```


