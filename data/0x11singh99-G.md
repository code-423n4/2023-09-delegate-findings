## Gas Optimizations

| Number |Issue|Instances|
|-|:-|:-:|
| [[G-01](#g-01-structs-can-be-packed-into-fewer-storage-slots)] | `Structs` can be packed into fewer storage slots. | 2 |
| [[G-02](#g-02-use-calldata-instead-of-memory-for-function-arguments-that-do-not-get-mutated)] | Use `calldata` instead of `memory` for function arguments that do not get mutated. | 1 | 
| [[G-03](#g-03-use-constants-instead-of-typeuintxmax)] | Use `constants` instead of type(uintx).max. | 7 |
| [[G-04](#g-04-no-need-to-explicitly-initialize-variables-with-default-values)] | No need to explicitly initialize variables with default values. | 7 |
| [[G-05](#g-05-duplicated-requireif-checks-should-be-refactored-to-a-modifier-or-function)] | Duplicated require()/if() checks should be refactored to a modifier or function | 2 |


## [G-01] Structs can be packed into fewer storage slots.

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs), we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

**_2 Instances - 2 Files_**

### Reduce uint type for `expiryLength` and pack  `expiryLength` with `tokenContract` of address type into a single storage slot to save 1 SLOT (~2000 gas).

`expiryLength` is storing time in seconds into it and `uint64` is more than enough for it so `expiryLength` type can be truncated to `uint64` from `uint256`. Since  `tokenContract` is address type which takes 20 bytes. So it can be packed with `expiryLength` of type `uint64` into one storage slot. 

```solidity
File : src/libraries/CreateOffererLib.sol

 98:     struct Order {
 99:        bytes32 rights;
100:        uint256 expiryLength;
101:        uint256 signerSalt;
102:        address tokenContract;
103:        CreateOffererEnums.ExpiryType expiryType;
104:        CreateOffererEnums.TargetToken targetToken;
105:    }

```
[98-105](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L98C2-L105C6)

```diff
File : src/libraries/CreateOffererLib.sol

   struct Order {
        bytes32 rights;
+       uint64 expiryLength;
+       address tokenContract;
-       uint256 expiryLength;
        uint256 signerSalt;
-       address tokenContract;
        CreateOffererEnums.ExpiryType expiryType;
        CreateOffererEnums.TargetToken targetToken;
    }
```

### Reduce uint type for `expiry` and pack  `expiry` with `tokenContract` of address type into a single storage slot to save 1 SLOT (~2000 gas).

`expiry` is storing time in seconds into it and `uint64` is more than enough for it so `expiry` type can be truncated to `uint64` from `uint256`. Since  `tokenContract` is address type which takes 20 bytes. So it can be packed with `expiry` of type `uint64` into one storage slot.

```solidity
File : src/libraries/DelegateTokenLib.sol

20:     struct DelegateInfo {
21:        address principalHolder;
22:        IDelegateRegistry.DelegationType tokenType;
23:        address delegateHolder;
24:        uint256 amount;
25:        address tokenContract;
26:        uint256 tokenId;
27:        bytes32 rights;
28:        uint256 expiry;
29:    }

```
[20-29](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L20C1-L29C6)

```diff
File : src/libraries/DelegateTokenLib.sol

  struct DelegateInfo {
        address principalHolder;
        IDelegateRegistry.DelegationType tokenType;
        address delegateHolder;
+       address tokenContract; 
+       uint64 expiry;       
        uint256 amount;
-       address tokenContract;
        uint256 tokenId;
        bytes32 rights;
-       uint256 expiry;
    }

```


## [G-02] Use calldata instead of memory for function arguments that do not get mutated.

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

**Note: These instances missed by bot report**

**_1 Instance - 1 File_**

```solidity
File : src/libraries/CreateOffererLib.sol

257: function createAndValidateDelegateTokenId(address delegateToken, uint256 createOrderHash, IDelegateTokenStructs.  DelegateInfo memory delegateInfo) internal {

```
[257](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L257)

```diff
File : src/libraries/CreateOffererLib.sol

- function createAndValidateDelegateTokenId(address delegateToken, uint256 createOrderHash, IDelegateTokenStructs.DelegateInfo memory delegateInfo) internal {
+ function createAndValidateDelegateTokenId(address delegateToken, uint256 createOrderHash, IDelegateTokenStructs.DelegateInfo calldata delegateInfo) internal {
        uint256 actualDelegateId = IDelegateToken(delegateToken).create(delegateInfo, createOrderHash);
        uint256 requestedDelegateId = DelegateTokenHelpers.delegateIdNoRevert(address(this), createOrderHash);
        if (actualDelegateId != requestedDelegateId) {
            revert CreateOffererErrors.DelegateTokenIdInvariant(requestedDelegateId, actualDelegateId);
        }
    }

```


## [G-03] Use constants instead of type(uintx).max.

In Solidity, type(uintX).max is used to represent the maximum value for an unsigned integer of X bits. This value is often used in contracts to represent an "infinity" value or indicate that a certain condition has not been met. However, using type(uintX).max can be expensive in terms of gas cost since it requires the compiler to perform a complex computation to determine the maximum value for an unsigned integer of X bits. 

**_7 Instances - 1 File_**

### To save gas , it's recommended to use `constants` instead of `type(uint256).max` to represent infinity or other large values.

```solidity
File : delegate-registry/src/DelegateRegistry.sol

213: ? type(uint256).max
214:             : _loadDelegationUint(Hashes.erc20Location(from, "", to, contract_), Storage.POSITIONS_AMOUNT);
215:        if (!Ops.or(rights == "", amount == type(uint256).max)) {
217:                ? type(uint256).max


232: ? type(uint256).max               
234:      if (!Ops.or(rights == "", amount == type(uint256).max)) {              
236:                ? type(uint256).max


372:  uint256 cleanUpper12Bytes = type(uint256).max << 160;
```
[213-217](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L213C17-L217C40),
[232-236](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L232C17-L236C40),
[372](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L372)

```diff
File : delegate-registry/src/DelegateRegistry.sol

contract DelegateRegistry is IDelegateRegistry {
    ...

+   uint256 constant MAX_UINT256 = 2**256 - 1;

     function checkDelegateForERC20
     ... {
        if (!_invalidFrom(from)) {
            amount = (_validateFrom(Hashes.allLocation(from, "", to), from) || _validateFrom(Hashes.contractLocation(from, "", to, contract_), from))
-               ? type(uint256).max
+               ? MAX_UINT256
                : _loadDelegationUint(Hashes.erc20Location(from, "", to, contract_), Storage.POSITIONS_AMOUNT);
-           if (!Ops.or(rights == "", amount == type(uint256).max)) {
+           if (!Ops.or(rights == "", amount == MAX_UINT256)) {
                uint256 rightsBalance = (_validateFrom(Hashes.allLocation(from, rights, to), from) || _validateFrom(Hashes.contractLocation(from, rights, to, contract_), from))
-                   ? type(uint256).max
+                   ? MAX_UINT256
                    : _loadDelegationUint(Hashes.erc20Location(from, rights, to, contract_), Storage.POSITIONS_AMOUNT);
                amount = Ops.max(rightsBalance, amount);
            }
        }
    }

    /// @inheritdoc IDelegateRegistry
    function checkDelegateForERC1155
     ... {
        if (!_invalidFrom(from)) {
            amount = (_validateFrom(Hashes.allLocation(from, "", to), from) || _validateFrom(Hashes.contractLocation(from, "", to, contract_), from))
-               ? type(uint256).max
+               ? MAX_UINT256
                : _loadDelegationUint(Hashes.erc1155Location(from, "", to, tokenId, contract_), Storage.POSITIONS_AMOUNT);
-           if (!Ops.or(rights == "", amount == type(uint256).max)) {
+           if (!Ops.or(rights == "", amount == MAX_UINT256)) {
                uint256 rightsBalance = (_validateFrom(Hashes.allLocation(from, rights, to), from) || _validateFrom(Hashes.contractLocation(from, rights, to, contract_), from))
-                   ? type(uint256).max
+                   ? MAX_UINT256
                    : _loadDelegationUint(Hashes.erc1155Location(from, rights, to, tokenId, contract_), Storage.POSITIONS_AMOUNT);
                amount = Ops.max(rightsBalance, amount);
            }
        }
    ...}

    ...
       function _updateFrom(bytes32 location, address from) internal {
        uint256 firstPacked = Storage.POSITIONS_FIRST_PACKED;
        uint256 cleanAddress = Storage.CLEAN_ADDRESS;
-       uint256 cleanUpper12Bytes = type(uint256).max << 160;
+       uint256 cleanUpper12Bytes = MAX_UINT256 << 160;
    ... }


```


## [G-04] No need to explicitly initialize variables with default values.

If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address, etc.). Explicitly initializing it with its default value is an anti-pattern and wastes gas. 

**_7 Instances - 1 File_**

### *`for (uint256 i = 0; i < numIterations; ++i)`*  should be replaced with: *`for (uint256 i; i < numIterations; ++i)`* 

```solidity
File : delegate-registry/src/DelegateRegistry.sol

 35: for (uint256 i = 0; i < data.length; ++i) {

275: for (uint256 i = 0; i < hashes.length; ++i) {  

312: for (uint256 i = 0; i < length; ++i) {

386: for (uint256 i = 0; i < hashesLength; ++i) {

393: for (uint256 i = 0; i < count; ++i) {

417: for (uint256 i = 0; i < hashesLength; ++i) {

423: for (uint256 i = 0; i < count; ++i) {                    

```
[35](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L35),
[275](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L275),
[312](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L312),
[386](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L386),
[393](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L393),
[417](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L417),
[423](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L423)

### Do not initialize `uint` variable with default value `0`

```diff
File : delegate-registry/src/DelegateRegistry.sol

-381: uint256 count = 0;
+381: uint256 count ;

-412: uint256 count = 0;
+412: uint256 count ;

```
[381](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L381),
[412](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L412)

## [G-05] Duplicated `require()/if()` checks should be refactored to a modifier or function

Duplicated require()/if() checks should be refactored to a modifier or function

**_2 Instances - 1 File_**

```solidity
File : src/DelegateToken.sol

100: if (delegateTokenHolder == address(0)) revert Errors.DelegateTokenHolderZero();
107: if (delegateTokenHolder == address(0)) revert Errors.DelegateTokenHolderZero();

```
[100](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L100),
[107](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L107)

