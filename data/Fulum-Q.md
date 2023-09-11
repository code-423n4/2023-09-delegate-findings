# [01] For loop on a storage mapping can lead to a DoS

On the `DelegateRegistry`, anyone can call one of these function: `delegateAll`, `delegateContract`, `delegateERC721`, `delegateERC20` and `delegateERC1155`.
The functions with a good parameters do many things and one of them it's adding a hash in the mapping pointing on a `to` value, everyone can add different hash on these mapping.
And because the two functions `getIncomingDelegations` and `getIncomingDelegationHashes` use for loop based on this mapping (which can have a very high length), this can lead to an out of gas on 
these two functions executions and cause a DoS.

### Proof Of Concept

[getIncomingDelegations](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L252-L253)
[for loop](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L380-L408)
[getIncomingDelegationHashes](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L262-L263)
[for loop](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L411-L427)


### Mitigation
You can add a mechanism/function to limit the number of hashes that can be written to the mapping for a `to` parameters.

# [02] Payable functions when the contract doesn't receive ETH

The following function below are `payable` but the function aren not designed to use ETH and if a user makes a mistake, these ETH are now stuck in the contract.
There should be a require(0 == msg.value) to ensure no Ether is being sent to the contract which using payable function.

### Proof Of Concept 

[multicall](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L31-L41)
[delegateAll](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L44-L60)
[delegateContract](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L63-L79)
[delegateERC721](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L82-L99)
[delegateERC20](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L102-L123)
[delegateERC1155](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L126-L148)

### Mitigation

There should be a require(0 == msg.value) to ensure no Ether is being sent to the contract which using payable function.
Or simply remove the `payable` on these functions.

# [03] Bad documentation on the notice NatSpec

Bad documentation on `CreateOfferer` for the notice parameter, the documentation is for `ratifyOrder` function and not for `generateOrder`.

### Proof Of Concept 

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/CreateOfferer.sol#L65

### Mitigation

--     * @notice Implementation of seaport contract offerer generateOrder
++     * @notice Implementation of seaport contract offerer ratifyOrder