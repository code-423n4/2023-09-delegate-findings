0. QA: Consider adding informational comments above each function to help with understanding purpose of function

1. QA: A floating pragma is normally not recommended, best practice is to use a specific range or fixed version

A floating pragma solidity version is probably not the greatest idea, it could invite unexpected issues in the future.

Floating pragma not recommended:
`pragma solidity ^0.8.21;`


2. LOW: DelegateRegistry.sol - Lack of address(0) check for the `to` address(delegate token buyer) in several functions.

https://github.com/delegatexyz/delegate-registry/blob/0e0ae905a6692b8f46e9b2a8518b0723396b239f/src/DelegateRegistry.sol#L44
https://github.com/delegatexyz/delegate-registry/blob/0e0ae905a6692b8f46e9b2a8518b0723396b239f/src/DelegateRegistry.sol#L63
https://github.com/delegatexyz/delegate-registry/blob/0e0ae905a6692b8f46e9b2a8518b0723396b239f/src/DelegateRegistry.sol#L82
https://github.com/delegatexyz/delegate-registry/blob/0e0ae905a6692b8f46e9b2a8518b0723396b239f/src/DelegateRegistry.sol#L102
https://github.com/delegatexyz/delegate-registry/blob/0e0ae905a6692b8f46e9b2a8518b0723396b239f/src/DelegateRegistry.sol#L126

Affected functions:
delegateAll(), delegateContract(), delegateERC721(), delegateERC20(), delegateERC1155(), 

Recommendation:

Add an address(0) check at beginning of the function logic:

```solidity
require(to != address(0), "Zero address not allowed");
```
OR
```solidity
if (to == address(0)) revert Error.ZeroAddress();
```


3. LOW: DelegateRegistry.sol - Inconsistency between two lines of code from two different functions in terms of implementation approach.

https://github.com/delegatexyz/delegate-registry/blob/0e0ae905a6692b8f46e9b2a8518b0723396b239f/src/DelegateRegistry.sol#L120
https://github.com/delegatexyz/delegate-registry/blob/0e0ae905a6692b8f46e9b2a8518b0723396b239f/src/DelegateRegistry.sol#L145

Summary:
The value of `amount` function variable/parameter on lines L120 & L145 is expected to be zero, according to the logic of the `if statement` block, but these two lines represent two different implementations for same idea.

L120:
```solidity
        } else if (loadedFrom == msg.sender) {
            _updateFrom(location, Storage.DELEGATION_REVOKED);
            _writeDelegation(location, Storage.POSITIONS_AMOUNT, amount); /// @audit this line uses `amount`
        }
```

VS

L145:
```solidity
        } else if (loadedFrom == msg.sender) {
            _updateFrom(location, Storage.DELEGATION_REVOKED);
            _writeDelegation(location, Storage.POSITIONS_AMOUNT, uint256(0)); /// @audit this line uses `uint256(0)`
        }
```

Recommendation:

Stick to one implementation approach, and using `uint256(0)` is probably better and more clear/explicit, eliminating any confusion as to the value of `amount`.


4. QA: DelegateRegistry::readSlot() & readSlots() - Should these external view functions be callable by just any arbitrary user, since it returns the contents of the storage locations via assembly?

https://github.com/delegatexyz/delegate-registry/blob/0e0ae905a6692b8f46e9b2a8518b0723396b239f/src/DelegateRegistry.sol#L300-L304


5. QA: RegistryHashes::erc1155Location() - Incorrect & misleading comment at L287. 

https://github.com/delegatexyz/delegate-registry/blob/cd0b3c17e8cc5222ebc55faa9b30b4f0acf55d8c/src/libraries/RegistryHashes.sol#L287

Corrected comment is: "Helper function to compute delegation location for ERC1155 delegation"
