# GAS OPTIMIZATIONS

##

## [G-1] Structs can be packed into fewer storage slots

#### Saves ``6000 GAS`` , ``3 SLOT``

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct.

Subsequent reads as well as writes have smaller gas savings.

### ``expiry`` can be uint96 instead of ``uint256`` :  Saves ``2000 GAS`` , ``1 SLOT``

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenLib.sol#L20-L29

A ``uint96`` can store values from ``0 to 2^96 - 1``, which is a very large range. However, it's important to note that Ethereum's ``block.timestamp`` is a ``Unix timestamp``, which represents time in seconds.
A ``uint96`` would overflow after approximately ``2,508,149,904,626,209 years`` when storing time in seconds.  This is an extremely long time frame, and it's highly unlikely that Ethereum or any blockchain system will remain unchanged for such an extended period.


```diff
FILE: 2023-09-delegate/src/libraries/DelegateTokenLib.sol

20: struct DelegateInfo {
21:        address principalHolder;
22:        IDelegateRegistry.DelegationType tokenType;
23:        address delegateHolder;
24:        uint256 amount;
25:        address tokenContract;
+ 28:        uint96 expiry;
26:        uint256 tokenId;
27:        bytes32 rights;
- 28:        uint256 expiry;
29:    }

```

### ``signerSalt``, ``expiryLength`` can be ``uint128`` instead of ``uint256`` : Saves ``4000 GAS`` , ``2 SLOT``

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L89-L105

In many blockchain protocols, the usage of ``salt`` often involves values within the range of ``uint32``, as they provide a sufficient numeric space for generating unique salts. Therefore, adopting a uint32 data type for the signerSalt field is a reasonable choice, as it aligns with common practices and conserves storage resources.

For the ``expiryLength`` field, which is intended to store the ``duration`` or ``length`` of an ``expiration period``, using ``uint128`` is more than adequate. This choice allows for a vast range of possible expiration lengths, accommodating a wide spectrum of use cases without incurring unnecessary storage overhead

```diff
FILE: 2023-09-delegate/src/libraries/CreateOffererLib.sol

89: struct Context {
90:        bytes32 rights;
+ 91:        uint128 signerSalt;
+ 92:        uint128 expiryLength;
- 91:        uint256 signerSalt;
- 92:        uint256 expiryLength;
93:        CreateOffererEnums.ExpiryType expiryType;
94:        CreateOffererEnums.TargetToken targetToken;
95:    }


98: struct Order {
99:        bytes32 rights;
- 100:        uint256 expiryLength;
- 101:        uint256 signerSalt;
+ 100:        uint128 expiryLength;
+ 101:        uint128 signerSalt;
102:        address tokenContract;
103:        CreateOffererEnums.ExpiryType expiryType;
104:        CreateOffererEnums.TargetToken targetToken;
105:    }

```

## [G-2] Remove or replace unused state variables

#### Saves ``20000 GAS ``

Saves a storage slot. If the variable is assigned a non-zero value, saves ``Gsset (20000 gas)``. If it's assigned a zero value, saves ``Gsreset (2900 gas)``. If the variable remains unassigned, there is no gas savings unless the variable is public, in which case the compiler-generated non-payable getter deployment cost is saved. If the state variable is overriding an interface's public function, mark the variable as constant or immutable so that it does not use a storage slot

```solidity
FILE: delegate-registry/src/DelegateRegistry.sol

18: mapping(bytes32 delegationHash => bytes32[5] delegationStorage) internal delegations;

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L18

##

## [G-3] State variables should be cached 

#### Saves ``300 GAS``, ``3 SLOD``

Caching of a state variable replaces each Gwarmaccess (100 gas) with a cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

### ``nonce.value``, ``receivers.targetTokenReceiver`` , ``receivers.fulfiller``  should be cached : Saves ``300 GAS``, ``3 SLOD``

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L185

```diff
FILE: 2023-09-delegate/src/libraries/CreateOffererLib.sol

+ uint256 value_ = nonce.value ;
- 185: if (nonce.value != contractNonce) revert CreateOffererErrors.InvalidContractNonce(nonce.value, contractNonce);
+ 185: if (value_ != contractNonce) revert CreateOffererErrors.InvalidContractNonce(value_, contractNonce);



+ address targetTokenReceiver_ = receivers.targetTokenReceiver ;
+ address fulfiller_ = receivers.fulfiller;
289: //slither-disable-start timestamp
300:        if (
            keccak256(
                abi.encode(
                    IDelegateTokenStructs.DelegateInfo({
                        tokenType: tokenType,
-                         principalHolder: decodedContext.targetToken == CreateOffererEnums.TargetToken.principal 
  ? receivers.targetTokenReceiver : receivers.fulfiller,
+                         principalHolder: decodedContext.targetToken == CreateOffererEnums.TargetToken.principal 
  ? targetTokenReceiver_  : fulfiller_ ,
-                        delegateHolder: decodedContext.targetToken == CreateOffererEnums.TargetToken.delegate ? receivers.targetTokenReceiver : receivers.fulfiller,
+                        delegateHolder: decodedContext.targetToken == CreateOffererEnums.TargetToken.delegate ? targetTokenReceiver_  : fulfiller_ ,
                        expiry: CreateOffererHelpers.calculateExpiry(decodedContext.expiryType, decodedContext.expiryLength),
                        rights: decodedContext.rights,
                        tokenContract: consideration.token,
                        tokenId: (tokenType != IDelegateRegistry.DelegationType.ERC20) ? consideration.identifier : 0,
                        amount: (tokenType != IDelegateRegistry.DelegationType.ERC721) ? consideration.amount : 0
                    })
                )
            ) != keccak256(abi.encode(IDelegateToken(delegateToken).getDelegateInfo(DelegateTokenHelpers.delegateIdNoRevert(address(this), identifier))))
        ) revert CreateOffererErrors.DelegateInfoInvariant();
```

##

## [G-4] Use assembly for loops to save gas

#### Saves ``2450 GAS`` for every iteration from ``7 instances ``

Assembly is more gas efficient for loops. Saves minimum ``350 GAS`` per iteration as per remix gas checks.

```solidity
FILE: Breadcrumbsdelegate-registry/src/DelegateRegistry.sol

- 35: for (uint256 i = 0; i < data.length; ++i) {
-                //slither-disable-next-line calls-loop,delegatecall-loop
-                (success, results[i]) = address(this).delegatecall(data[i]);
-                if (!success) revert MulticallFailed();
-            }

+ assembly {
+    // Load the length of the data array
+    let dataSize := mload(data)
+
+    // Initialize the results array
+    let results := mload(0x40)
+    mstore(results, dataSize)
+
+    // Initialize a counter (i) to zero
+    let i := 0
+
+    for { } lt(i, dataSize) { } {
+        // Start loop
+
+        // Load the next calldata from the data array
+        let calldataPtr := add(add(data, 0x20), mul(i, 0x20))
+        let calldataSize := mload(calldataPtr)
+
+        // Perform delegatecall
+        let success := delegatecall(gas(), address(), add(calldataPtr, 0x04), calldataSize, 0, 0)
+
+        // Store the result and check for success
+        if iszero(success) {
+            // Revert if delegatecall fails
+            revert(0, 0)
+        }
+        mstore(add(results, mul(i, 0x20)), success)
+
+        // Increment the counter
+        i := add(i, 1)
+
+        // End loop
+    }
+
+    // results contains the success status of each delegatecall
+
+ }

275:  for (uint256 i = 0; i < hashes.length; ++i) {

312: for (uint256 i = 0; i < length; ++i) {
                tempLocation = locations[i];
                assembly {
                    tempValue := sload(tempLocation)
                }
                contents[i] = tempValue;
            }

386: for (uint256 i = 0; i < hashesLength; ++i) {
                hash = hashes[i];
                if (_invalidFrom(_loadFrom(Hashes.location(hash)))) continue;
                filteredHashes[count++] = hash;
            }

393:  for (uint256 i = 0; i < count; ++i) {
                hash = filteredHashes[i];
                location = Hashes.location(hash);
                (address from, address to, address contract_) = _loadDelegationAddresses(location);
                delegations_[i] = Delegation({
                    type_: Hashes.decodeType(hash),
                    to: to,
                    from: from,
                    rights: _loadDelegationBytes32(location, Storage.POSITIONS_RIGHTS),
                    amount: _loadDelegationUint(location, Storage.POSITIONS_AMOUNT),
                    contract_: contract_,
                    tokenId: _loadDelegationUint(location, Storage.POSITIONS_TOKEN_ID)
                });
            }

417: for (uint256 i = 0; i < hashesLength; ++i) {
                hash = hashes[i];
                if (_invalidFrom(_loadFrom(Hashes.location(hash)))) continue;
                filteredHashes[count++] = hash;
            }

423: for (uint256 i = 0; i < count; ++i) {
                validHashes[i] = filteredHashes[i];
            }

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L423-L425



##

## [G-5] Don't initialize default values to variables to reduce gas 

Saves ``13 GAS`` for local variable and ``2000 GAS`` for state variable 

```solidity
FILE: Breadcrumbsdelegate-registry/src/DelegateRegistry.sol

35: for (uint256 i = 0; i < data.length; ++i) {
275: for (uint256 i = 0; i < hashes.length; ++i) {
312: for (uint256 i = 0; i < length; ++i) {
386: for (uint256 i = 0; i < hashesLength; ++i) {
393: for (uint256 i = 0; i < count; ++i) {
417: for (uint256 i = 0; i < hashesLength; ++i) {
423: for (uint256 i = 0; i < count; ++i) {

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L35

##

## [G-6] Use constants instead of type(uintX).max to avoid calculating every time 

```solidity
FILE: delegate-registry/src/DelegateRegistry.sol

213:    ? type(uint256).max
215: if (!Ops.or(rights == "", amount == type(uint256).max)) {
217: ? type(uint256).max
232: ? type(uint256).max
234: if (!Ops.or(rights == "", amount == type(uint256).max)) {
236: ? type(uint256).max
372: uint256 cleanUpper12Bytes = type(uint256).max << 160;

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L213

##

## [G-7] For same condition checks use modifiers 

```solidity
File: delegate-registry/src/DelegateRegistry.sol

56: } else if (loadedFrom == msg.sender) {
75: } else if (loadedFrom == msg.sender) {
85: } else if (loadedFrom == msg.sender) {
115:  } else if (loadedFrom == msg.sender) {
118: } else if (loadedFrom == msg.sender) {
140: } else if (loadedFrom == msg.sender) {
143: } else if (loadedFrom == msg.sender) {

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L75C9-L75C47

##

## [G-8] Declare the variables outside the loop

Per iterations saves ``26 GAS``

```diff
FILE: delegate-registry/src/DelegateRegistry.sol

+                 bytes32 location ;
+                 address from ;
 for (uint256 i = 0; i < hashes.length; ++i) {
-                 bytes32 location = Hashes.location(hashes[i]);
-                 address from = _loadFrom(location);
+                location = Hashes.location(hashes[i]);
+                from = _loadFrom(location);
                if (_invalidFrom(from)) {
                    delegations_[i] = Delegation({type_: DelegationType.NONE, to: address(0), from: address(0), rights: "", amount: 0, contract_: address(0), tokenId: 0});
                } else {
                    (, address to, address contract_) = _loadDelegationAddresses(location);
                    delegations_[i] = Delegation({
                        type_: Hashes.decodeType(hashes[i]),
                        to: to,
                        from: from,
                        rights: _loadDelegationBytes32(location, Storage.POSITIONS_RIGHTS),
                        amount: _loadDelegationUint(location, Storage.POSITIONS_AMOUNT),
                        contract_: contract_,
                        tokenId: _loadDelegationUint(location, Storage.POSITIONS_TOKEN_ID)
                    });

```












