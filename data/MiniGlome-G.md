## Gas Optimizations
| |Issue|Instances|
|-|:-|:-:| 
| [GAS-01] | 0 amount variable can be replaced by uint256(0) | 1 | 
| [GAS-02] | Check on function arguments should be put before state modifications | 1 | 
| [GAS-03] | Avoid useless type casting | 1 | 

### [GAS-01] 0 amount variable can be replaced by uint256(0)
The `amount` variable is `0` in the following `else if` statement, so it can be replaced by `uint256(0)`.
*Instance (1)*:
```solidity
File: DelegateRegistry.sol

106: if (amount != 0) {
           // [...]
     } else if (loadedFrom == msg.sender) {
          _updateFrom(location, Storage.DELEGATION_REVOKED);
          _writeDelegation(location, Storage.POSITIONS_AMOUNT, amount); //@audit [GAS] amount can be replaced by uint256(0)
     }
```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L120


### [GAS-02] Check on function arguments should be put before state modifications
Checks on function arguments should be put before any contract logic and state modifications
*Instance (1)*:
```solidity
File: DelegateToken.sol

353: function withdraw(uint256 delegateTokenId) external nonReentrant {
        bytes32 registryHash = StorageHelpers.readRegistryHash(delegateTokenInfo, delegateTokenId);
        StorageHelpers.writeRegistryHash(delegateTokenInfo, delegateTokenId, bytes32(StorageHelpers.ID_USED));
        // Sets registry pointer to used flag
        StorageHelpers.revertNotMinted(registryHash, delegateTokenId); //@audit [GAS] can be put before
```
https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/DelegateToken.sol#L357

### [GAS-03] Avoid useless type casting
*Instance (1)*:
```solidity
File: DelegateTokenRegistryHelpers.sol

194: function transferERC20(
        address delegateRegistry,
        bytes32 registryHash,
        address from,
        bytes32 newRegistryHash,
        address to,
        uint256 underlyingAmount,
        bytes32 underlyingRights,
        address underlyingContract
    ) internal {
        if (
            IDelegateRegistry(delegateRegistry).delegateERC20(
                from, underlyingContract, underlyingRights, calculateDecreasedAmount(delegateRegistry, registryHash, underlyingAmount)
            ) == bytes32(registryHash) //@audit [GAS] useless casting, registryHash is already of type bytes32

```
https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenRegistryHelpers.sol#L207
